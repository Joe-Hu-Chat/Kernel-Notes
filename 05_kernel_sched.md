

# wait

## DEFINE_WAIT_FUNC

![image-20240605145613635](./.05_kernel_sched/image-20240605145613635.png)

### `struct` wait_queue_entry

![image-20240605150940300](./.05_kernel_sched/image-20240605150940300.png)

## woken_wake_function

![image-20240605145350217](./.05_kernel_sched/image-20240605145350217.png)

### default_wake_function

![image-20240605151531364](./.05_kernel_sched/image-20240605151531364.png)

### try_to_wake_up

```c
/**
 * try_to_wake_up - wake up a thread
 * @p: the thread to be awakened
 * @state: the mask of task states that can be woken
 * @wake_flags: wake modifier flags (WF_*)
 *
 * Conceptually does:
 *
 *   If (@state & @p->state) @p->state = TASK_RUNNING.
 *
 * If the task was not queued/runnable, also place it back on a runqueue.
 *
 * This function is atomic against schedule() which would dequeue the task.
 *
 * It issues a full memory barrier before accessing @p->state, see the comment
 * with set_current_state().
 *
 * Uses p->pi_lock to serialize against concurrent wake-ups.
 *
 * Relies on p->pi_lock stabilizing:
 *  - p->sched_class
 *  - p->cpus_ptr
 *  - p->sched_task_group
 * in order to do migration, see its use of select_task_rq()/set_task_cpu().
 *
 * Tries really hard to only take one task_rq(p)->lock for performance.
 * Takes rq->lock in:
 *  - ttwu_runnable()    -- old rq, unavoidable, see comment there;
 *  - ttwu_queue()       -- new rq, for enqueue of the task;
 *  - psi_ttwu_dequeue() -- much sadness :-( accounting will kill us.
 *
 * As a consequence we race really badly with just about everything. See the
 * many memory barriers and their comments for details.
 *
 * Return: %true if @p->state changes (an actual wakeup was done),
 *	   %false otherwise.
 */
static int
try_to_wake_up(struct task_struct *p, unsigned int state, int wake_flags)
{
	unsigned long flags;
	int cpu, success = 0;

	preempt_disable();
	if (p == current) {
		/*
		 * We're waking current, this means 'p->on_rq' and 'task_cpu(p)
		 * == smp_processor_id()'. Together this means we can special
		 * case the whole 'p->on_rq && ttwu_runnable()' case below
		 * without taking any locks.
		 *
		 * In particular:
		 *  - we rely on Program-Order guarantees for all the ordering,
		 *  - we're serialized against set_special_state() by virtue of
		 *    it disabling IRQs (this allows not taking ->pi_lock).
		 */
		if (!(p->state & state))
			goto out;

		success = 1;
		trace_sched_waking(p);
		p->state = TASK_RUNNING;
		trace_sched_wakeup(p);
		goto out;
	}

	/*
	 * If we are going to wake up a thread waiting for CONDITION we
	 * need to ensure that CONDITION=1 done by the caller can not be
	 * reordered with p->state check below. This pairs with smp_store_mb()
	 * in set_current_state() that the waiting thread does.
	 */
	raw_spin_lock_irqsave(&p->pi_lock, flags);
	smp_mb__after_spinlock();
	if (!(p->state & state))
		goto unlock;

	trace_sched_waking(p);

	/* We're going to change ->state: */
	success = 1;

	/*
	 * Ensure we load p->on_rq _after_ p->state, otherwise it would
	 * be possible to, falsely, observe p->on_rq == 0 and get stuck
	 * in smp_cond_load_acquire() below.
	 *
	 * sched_ttwu_pending()			try_to_wake_up()
	 *   STORE p->on_rq = 1			  LOAD p->state
	 *   UNLOCK rq->lock
	 *
	 * __schedule() (switch to task 'p')
	 *   LOCK rq->lock			  smp_rmb();
	 *   smp_mb__after_spinlock();
	 *   UNLOCK rq->lock
	 *
	 * [task p]
	 *   STORE p->state = UNINTERRUPTIBLE	  LOAD p->on_rq
	 *
	 * Pairs with the LOCK+smp_mb__after_spinlock() on rq->lock in
	 * __schedule().  See the comment for smp_mb__after_spinlock().
	 *
	 * A similar smb_rmb() lives in try_invoke_on_locked_down_task().
	 */
	smp_rmb();
	if (READ_ONCE(p->on_rq) && ttwu_runnable(p, wake_flags))
		goto unlock;

#ifdef CONFIG_SMP
	/*
	 * Ensure we load p->on_cpu _after_ p->on_rq, otherwise it would be
	 * possible to, falsely, observe p->on_cpu == 0.
	 *
	 * One must be running (->on_cpu == 1) in order to remove oneself
	 * from the runqueue.
	 *
	 * __schedule() (switch to task 'p')	try_to_wake_up()
	 *   STORE p->on_cpu = 1		  LOAD p->on_rq
	 *   UNLOCK rq->lock
	 *
	 * __schedule() (put 'p' to sleep)
	 *   LOCK rq->lock			  smp_rmb();
	 *   smp_mb__after_spinlock();
	 *   STORE p->on_rq = 0			  LOAD p->on_cpu
	 *
	 * Pairs with the LOCK+smp_mb__after_spinlock() on rq->lock in
	 * __schedule().  See the comment for smp_mb__after_spinlock().
	 *
	 * Form a control-dep-acquire with p->on_rq == 0 above, to ensure
	 * schedule()'s deactivate_task() has 'happened' and p will no longer
	 * care about it's own p->state. See the comment in __schedule().
	 */
	smp_acquire__after_ctrl_dep();

	/*
	 * We're doing the wakeup (@success == 1), they did a dequeue (p->on_rq
	 * == 0), which means we need to do an enqueue, change p->state to
	 * TASK_WAKING such that we can unlock p->pi_lock before doing the
	 * enqueue, such as ttwu_queue_wakelist().
	 */
	p->state = TASK_WAKING;

	/*
	 * If the owning (remote) CPU is still in the middle of schedule() with
	 * this task as prev, considering queueing p on the remote CPUs wake_list
	 * which potentially sends an IPI instead of spinning on p->on_cpu to
	 * let the waker make forward progress. This is safe because IRQs are
	 * disabled and the IPI will deliver after on_cpu is cleared.
	 *
	 * Ensure we load task_cpu(p) after p->on_cpu:
	 *
	 * set_task_cpu(p, cpu);
	 *   STORE p->cpu = @cpu
	 * __schedule() (switch to task 'p')
	 *   LOCK rq->lock
	 *   smp_mb__after_spin_lock()		smp_cond_load_acquire(&p->on_cpu)
	 *   STORE p->on_cpu = 1		LOAD p->cpu
	 *
	 * to ensure we observe the correct CPU on which the task is currently
	 * scheduling.
	 */
	if (smp_load_acquire(&p->on_cpu) &&
	    ttwu_queue_wakelist(p, task_cpu(p), wake_flags | WF_ON_CPU))
		goto unlock;

	/*
	 * If the owning (remote) CPU is still in the middle of schedule() with
	 * this task as prev, wait until its done referencing the task.
	 *
	 * Pairs with the smp_store_release() in finish_task().
	 *
	 * This ensures that tasks getting woken will be fully ordered against
	 * their previous state and preserve Program Order.
	 */
	smp_cond_load_acquire(&p->on_cpu, !VAL);

	cpu = select_task_rq(p, p->wake_cpu, SD_BALANCE_WAKE, wake_flags);
	if (task_cpu(p) != cpu) {
		if (p->in_iowait) {
			delayacct_blkio_end(p);
			atomic_dec(&task_rq(p)->nr_iowait);
		}

		wake_flags |= WF_MIGRATED;
		psi_ttwu_dequeue(p);
		set_task_cpu(p, cpu);
	}
#else
	cpu = task_cpu(p);
#endif /* CONFIG_SMP */

	ttwu_queue(p, cpu, wake_flags);
unlock:
	raw_spin_unlock_irqrestore(&p->pi_lock, flags);
out:
	if (success)
		ttwu_stat(p, task_cpu(p), wake_flags);
	preempt_enable();

	return success;
}
```



## add_wait_queue

![image-20240605145043244](./.05_kernel_sched/image-20240605145043244.png)

### struct wait_queue_head

![image-20240605151054500](./.05_kernel_sched/image-20240605151054500.png)

### __add_wait_queue

```c
static inline void __add_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
	list_add(&wq_entry->entry, &wq_head->head);
}
```



## wait_woken

![image-20240605145205719](./.05_kernel_sched/image-20240605145205719.png)

`schedule_timeout` will sleep until timeout, or woken up by signals if `TASK_INTERRUPTIBLE` flag was set by `set_current_state()`.  See `preemption` doc for explanation about scheduling.



# wait queue



![image-20240605154616521](./.05_kernel_sched/image-20240605154616521.png)

## waitqueue_active

![image-20240605154913879](./.05_kernel_sched/image-20240605154913879.png)

## __wake_up

![image-20240605154517268](./.05_kernel_sched/image-20240605154517268.png)



# sched_tick

## struct tick_sched

![image-20240726152911907](./.05_kernel_sched/image-20240726152911907.png)

![image-20240726152955886](./.05_kernel_sched/image-20240726152955886.png)

### struct hrtimer                                                                                                                                                                                                                                        

![image-20240726154003359](./.05_kernel_sched/image-20240726154003359.png)

#### struct hrtimer_clock_base

![image-20240726162402280](./.05_kernel_sched/image-20240726162402280.png)

## tick_nohz_activate

![image-20240726160014318](./.05_kernel_sched/image-20240726160014318.png)

Nohz mode, also known as "tickless" mode, is a feature in the Linux kernel that aims to reduce or eliminate periodic timer interrupts (ticks) on CPUs under certain conditions. This feature is designed to improve energy efficiency and reduce system jitter, especially for systems with real-time or low-latency requirements.

## tick_init_highres

Only gets called in function `hrtimer_switch_to_hres`, which in turn only gets called by `hrtimer_run_queues`. Then finally will be called in `update_process_times`:

![image-20240726171706686](./.05_kernel_sched/image-20240726171706686.png)

![image-20240726165858229](./.05_kernel_sched/image-20240726165858229.png)

### tick_switch_to_oneshot

The handler `hrtimer_interrupt` will be added to the `clock_event_device` in `tick_cpu_device`.

![image-20240726170359664](./.05_kernel_sched/image-20240726170359664.png)

### struct tick_device

![image-20240726170845142](./.05_kernel_sched/image-20240726170845142.png)

#### struct clock_event_device

![image-20240726171001702](./.05_kernel_sched/image-20240726171001702.png)

![image-20240726171038197](./.05_kernel_sched/image-20240726171038197.png)



### clockevents_switch_state

![image-20240726170449317](./.05_kernel_sched/image-20240726170449317.png)



## hrtimer



### hrtimer_init

![image-20240726155359942](./.05_kernel_sched/image-20240726155359942.png)

### __hrtimer_init

![image-20240726163117779](./.05_kernel_sched/image-20240726163117779.png)

#### timerqueue_init

![image-20240726163159472](./.05_kernel_sched/image-20240726163159472.png)

### hrtimer_start

![image-20240726161525387](./.05_kernel_sched/image-20240726161525387.png)

### hrtimer_set_expires

![image-20240726155514771](./.05_kernel_sched/image-20240726155514771.png)

### hrtimer_start_expires

![image-20240726155609539](./.05_kernel_sched/image-20240726155609539.png)

#### struct ktime_t

![image-20240726160709720](./.05_kernel_sched/image-20240726160709720.png)

### hrtimer get expires

![image-20240726161128169](./.05_kernel_sched/image-20240726161128169.png)

### hrtimer_start_range_ns

![image-20240726161614880](./.05_kernel_sched/image-20240726161614880.png)

### __hrtimer_start_range_ns

![image-20240726161705518](./.05_kernel_sched/image-20240726161705518.png)



#### switch_hrtimer_base

![image-20240726162105303](./.05_kernel_sched/image-20240726162105303.png)

![image-20240726162123590](./.05_kernel_sched/image-20240726162123590.png)

#### enqueue_hrtimer

![image-20240726161749392](./.05_kernel_sched/image-20240726161749392.png)

#### timerqueue_add

![image-20240726161830155](./.05_kernel_sched/image-20240726161830155.png)



### tick_setup_periodic

![image-20240730170213222](./.05_kernel_sched/image-20240730170213222.png)

### tick_set_periodic_handler

![image-20240730170126730](./.05_kernel_sched/image-20240730170126730.png)

### tick_handle_periodic

![image-20240730152814206](./.05_kernel_sched/image-20240730152814206.png)

#### tick_periodic

![image-20240730152725499](./.05_kernel_sched/image-20240730152725499.png)



### hrtimer_run_queues

![image-20240730165858263](./.05_kernel_sched/image-20240730165858263.png)

### hrtimer_switch_to_hres

![image-20240730153617960](./.05_kernel_sched/image-20240730153617960.png)

### tick_setup_sched_timer

![image-20240730153037846](./.05_kernel_sched/image-20240730153037846.png)

### tick_sched_timer

![image-20240730152951207](./.05_kernel_sched/image-20240730152951207.png)



### hrtimer_run_queues

![image-20240730153426877](./.05_kernel_sched/image-20240730153426877.png)

### tick_check_oneshot_change

![image-20240730153318710](./.05_kernel_sched/image-20240730153318710.png)

### tick_nohz_switch_to_nohz

![image-20240730153202579](./.05_kernel_sched/image-20240730153202579.png)

### tick_nohz_handler

![image-20240730152912535](./.05_kernel_sched/image-20240730152912535.png)

#### tick_sched_handle

![image-20240730152650914](./.05_kernel_sched/image-20240730152650914.png)

### update_process_times

![image-20240730152512186](./.05_kernel_sched/image-20240730152512186.png)

### run_local_timers

![image-20240730152342578](./.05_kernel_sched/image-20240730152342578.png)

### hrtimer_run_queues

![image-20240730152220320](./.05_kernel_sched/image-20240730152220320.png)



### hrtimer_switch_to_hres

![image-20240730152136902](./.05_kernel_sched/image-20240730152136902.png)





### hrtimer_forward

![image-20240726155743907](./.05_kernel_sched/image-20240726155743907.png)

![image-20240726155800490](./.05_kernel_sched/image-20240726155800490.png)



### __hrtimer_peek_ahead_timers

![image-20240726170054953](./.05_kernel_sched/image-20240726170054953.png)

### hrtimer_interrupt

event_handler

![image-20240726163615914](./.05_kernel_sched/image-20240726163615914.png)

![image-20240726163641385](./.05_kernel_sched/image-20240726163641385.png)

![image-20240726163708673](./.05_kernel_sched/image-20240726163708673.png)

![image-20240726163730652](./.05_kernel_sched/image-20240726163730652.png)



### __hrtimer_run_queues

![image-20240726164040894](./.05_kernel_sched/image-20240726164040894.png)

### __run_hrtimer

![image-20240726164337846](./.05_kernel_sched/image-20240726164337846.png)

![image-20240726164438537](./.05_kernel_sched/image-20240726164438537.png)

![image-20240726164249084](./.05_kernel_sched/image-20240726164249084.png)



## hrtimers_init

This function is called in `start_kernel` function.

![image-20240726165251062](./.05_kernel_sched/image-20240726165251062.png)

### hrtimers_prepare_cpu

![image-20240726165221469](./.05_kernel_sched/image-20240726165221469.png)





# timer initialization

All the timers will be declared by macro `TIMER_OF_DECLARE`, and then get initialized in `timer_probe` function.

## TIMER_OF_DECLARE

![image-20240726100437598](./.05_kernel_sched/image-20240726100437598.png)

![image-20240726100521376](./.05_kernel_sched/image-20240726100521376.png)

![image-20240726110520402](./.05_kernel_sched/image-20240726110520402.png)

![image-20240726100551008](./.05_kernel_sched/image-20240726100551008.png)

Finally, this macro expands to:

```c
#define TIMER_OF_DECLARE(name, compat, fn)				as
static const struct of_device_id __of_table_##name		\
		__used __section("__" "timer" "_of_table")		\
    	 = { .compatible = compat,						\
             .data = (fn == (fn_type)NULL) ? fn : fn }	
```

Then the `struct` is defined to link to `__timer_of_table` sections:

```c
typedef int (*of_init_fn_1_ret)(struct device_node *);
#define TIMER_OF_DECLARE(name, compat, fn)				as
static const struct of_device_id __of_table_##name		\
		__used __section("__timer_of_table")		\
    	 = { .compatible = compat,						\
             .data = (fn == (of_init_fn_1_ret)NULL) ? fn : fn }	
```

## time_init

This function is called in `start_kernel` function.

![image-20240726142307329](./.05_kernel_sched/image-20240726142307329.png)

### timer_probe

The symbol `__timer_of_table` here, is set to pointing to the `__timer_of_table` section in linker script file.

![image-20240726110956187](./.05_kernel_sched/image-20240726110956187.png)

#### for_each_matching_node_and_match

![image-20240726144056850](./.05_kernel_sched/image-20240726144056850.png)

![image-20240731142757367](./.05_kernel_sched/image-20240731142757367.png)

#### __of_match_node

![image-20240731142644284](./.05_kernel_sched/image-20240731142644284.png)

#### __of_device_is_compatible

![image-20240731142844314](./.05_kernel_sched/image-20240731142844314.png)

![image-20240731142923863](./.05_kernel_sched/image-20240731142923863.png)

#### __timer_of_table

![image-20240726111654808](./.05_kernel_sched/image-20240726111654808.png)

![image-20240726111725988](./.05_kernel_sched/image-20240726111725988.png)

![image-20240726112307076](./.05_kernel_sched/image-20240726112307076.png)

Here `TIMER_OF_TABALES()` expands to `_OF_TABLE_0(timer)`. 

![image-20240726112123023](./.05_kernel_sched/image-20240726112123023.png)

Then:

```c
#define _OF_TABLE_0(timer)
		. = ALIGN(8);						\
        __timer_of_table = .;				\
        KEEP(*(__timer_of_table));			\
        KEEP(*(__timer_of_table_end))
```

## timer drivers

Corresponding drivers should be configured properly, and then the "compatible" strings in device tree can be used to drivers pointed in the `__timer_of_table` section.

![image-20240731142338557](./.05_kernel_sched/image-20240731142338557.png)

### clint_timer_init_dt

![image-20240809151904827](./.05_kernel_sched/image-20240809151904827.png)

![image-20240731170450857](./.05_kernel_sched/image-20240731170450857.png)

![image-20240731112520134](./.05_kernel_sched/image-20240731112520134.png)

The macro `TIMER_OF_DECLARE` will expand here: 

```c
typedef int (*of_init_fn_1_ret)(struct device_node *);
#define TIMER_OF_DECLARE(clint_timer, "riscv,clint0", fn)
// will expand as the struct below
static const struct of_device_id __of_table_clint_timer
		__used __section("__timer_of_table")
    	 = { .compatible = "riscv,clint0",
             .data = (fn == (of_init_fn_1_ret)NULL) ? fn : fn }	
// fn = clint_timer_init_dt;
```



### riscv_timer_init_dt

![image-20240726095107294](./.05_kernel_sched/image-20240726095107294.png)

![image-20240726095145081](./.05_kernel_sched/image-20240726095145081.png)

Here this macro expands as:

```c
typedef int (*of_init_fn_1_ret)(struct device_node *);
#define TIMER_OF_DECLARE(riscv_timer, "riscv", fn)
// will expand as the struct below
static const struct of_device_id __of_table_riscv_timer
		__used __section("__timer_of_table")
    	 = { .compatible = "riscv",
             .data = (fn == (of_init_fn_1_ret)NULL) ? fn : fn }	
// fn = riscv_timer_init_dt;
```

### cpuhp_setup_state

![image-20240726144906265](./.05_kernel_sched/image-20240726144906265.png)

### __cpuhp_setup_state

![image-20240726144936621](./.05_kernel_sched/image-20240726144936621.png)

### __cpuhp_setup_state_cpuslocked

![image-20240726145043590](./.05_kernel_sched/image-20240726145043590.png)

![image-20240726145244184](./.05_kernel_sched/image-20240726145244184.png)

#### cpuhp_store_callbacks

`cpuhp_step`

![image-20240726145549321](./.05_kernel_sched/image-20240726145549321.png)

#### cpuhp_issue_call

![image-20240726145830895](./.05_kernel_sched/image-20240726145830895.png)

#### cpuhp_invoke_ap_callback

![image-20240726150629826](./.05_kernel_sched/image-20240726150629826.png)

![image-20240726150643993](./.05_kernel_sched/image-20240726150643993.png)

#### cpuhp_invoke_callback

![image-20240726150812819](./.05_kernel_sched/image-20240726150812819.png)

![image-20240726150902528](./.05_kernel_sched/image-20240726150902528.png)

![image-20240726150923700](./.05_kernel_sched/image-20240726150923700.png)

## clock_event_device



### clint_timer_starting_cpu

![image-20240731151336322](./.05_kernel_sched/image-20240731151336322.png)

#### clint_clock_event

![image-20240731151434548](./.05_kernel_sched/image-20240731151434548.png)

#### clint_clock_next_event

![image-20240731151517254](./.05_kernel_sched/image-20240731151517254.png)

### riscv_timer_starting_cpu

![image-20240726094912939](./.05_kernel_sched/image-20240726094912939.png)

#### riscv_clock_event

![image-20240726095919446](./.05_kernel_sched/image-20240726095919446.png)

#### riscv_clock_next_event

![image-20240726100056468](./.05_kernel_sched/image-20240726100056468.png)

### clockevents_config_and_register

![image-20240726094630152](./.05_kernel_sched/image-20240726094630152.png)

### clockevents_register_device

![image-20240726094416322](./.05_kernel_sched/image-20240726094416322.png)

#### __ clockevents_notify_released

This function is not relevant here, but doing similar stuff.

![image-20240726094450401](./.05_kernel_sched/image-20240726094450401.png)

## tick_cpu_device

tick devices

![image-20240726170845142](./.05_kernel_sched/image-20240726170845142.png)

### tick_device: tick_cpu_device

![image-20240730144911638](./.05_kernel_sched/image-20240730144911638.png)



### tick_check_new_device

The `riscv_clock_event` will be added to `tick_cpu_device` here. The `tick_cpu_device` is the `tick_device` for every CPU.

![image-20240730181349621](./.05_kernel_sched/image-20240730181349621.png)

#### __ tick_install_replacement

![image-20240726094235003](./.05_kernel_sched/image-20240726094235003.png)

### tick_setup_device

With a new `clock_event_device` added to the `tick_device`, the`clock_event_device`.`event_handler`  will normally be executed in the timer interrupt handling routine in the driver. The function for **clint timer** is `clint_timer_interrupt`.

![image-20240726094025905](./.05_kernel_sched/image-20240726094025905.png)

![image-20240730181229118](./.05_kernel_sched/image-20240730181229118.png)

#### __ tick_resume_local

![image-20240726094143751](./.05_kernel_sched/image-20240726094143751.png)

## clock_event_device

![image-20240726095749586](./.05_kernel_sched/image-20240726095749586.png)

![image-20240726095805666](./.05_kernel_sched/image-20240726095805666.png)

### clint_timer_interrupt

This one is currently used by setting "riscv,clint0" compatible in device tree.

![image-20240730144228614](./.05_kernel_sched/image-20240730144228614.png)

#### clint_clock_event

![image-20240730144311044](./.05_kernel_sched/image-20240730144311044.png)

### riscv_timer_interrupt

![image-20240730144517807](./.05_kernel_sched/image-20240730144517807.png)

#### riscv_clock_event

![image-20240730144627057](./.05_kernel_sched/image-20240730144627057.png)



# jiffies

Is the number of system ticks. The system tick frequency is set by `CONFIG_HZ`, while 100 means ticking 100 times in a second. The tick period is 10ms for `CONFIG_HZ=100`, as `tick = 1s / HZ`.



# time slice

