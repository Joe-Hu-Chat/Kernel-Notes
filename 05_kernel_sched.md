通用调度器目标：

- 交互任务响应快
- 批处理任务吞吐量大
- SMP高效多核，高cache和TLB命中率



# task scheduler

An example for Data structure used for **fair** scheduler: 

![image-20250402221914974](./.05_kernel_sched/image-20250402221914974.png)

Different kinds of processes:

- **Real-time Process**: very high priority, response time smaller than Interactive Process (Real-time Processes nowadays are processes that actually have a Deadline for their completion time-windows, so they are in the **DL** schedule class)

- **Interactive Process**: high priority (Interactive Processes need a "real-time" response, so they are classified in **RT** schedule class at the very early stage of OS development)
- **Batch Process**: long time slice, low priority, falls into **fair** schedule class



## scheduler entity

Sched Objects: （调度实体）
```
- struct sched_entity se; # CFS
- struct sched_rt_entity rt; # RT
- struct sched_dl_entity dl; # DL (DeadLine)
```



### struct sched_dl_entity

`/* file: include/linux/sched.h */`

![image-20250508111647253](./.05_kernel_sched/image-20250508111647253.png)

![image-20250504191814475](./.05_kernel_sched/image-20250504191814475.png)

![image-20250504191839903](./.05_kernel_sched/image-20250504191839903.png)

![image-20250504191855862](./.05_kernel_sched/image-20250504191855862.png)



### struct sched_rt_entity

![image-20250506201022391](./.05_kernel_sched/image-20250506201022391.png)



### struct sched_entity

`/* file: include/linux/sched.h */`

![image-20250509151037471](./.05_kernel_sched/image-20250509151037471.png)

![image-20250504191559479](./.05_kernel_sched/image-20250504191559479.png)



### struct sched_ext_entity

`/* include/linux/sched/ext.h */`

![image-20250506201512882](./.05_kernel_sched/image-20250506201512882.png)

![image-20250506201713417](./.05_kernel_sched/image-20250506201713417.png)



## sched_class

sched_class:（调度类）

- `stop_sched_class` # In SMP architecture, used to stop other task on a CPU, then **stop the CPU**
- `dl_sched_class` # Used for tasks needed to finish in a **time window**, highest priority for user tasks
- `rt_sched_class` # Real-time tasks, related to **user interaction**, priority lover than DL
- `fair_sched_class` # Used for most tasks, to fairly allocate CPU time to tasks, like **CFS**
- `idle_sched_class` # Similar to stop class, only for kernel threads, lowest priority, to lower power consumption when **CPU idle**



`/* file: include/asm-generic/vmlinux.lds.h */`

![image-20250402175719236](./.05_kernel_sched/image-20250402175719236.png)

The implementation above is in v5.10 kernel code. The newest layout changes and the range names are changed to `highest` and `lowest` too, to make it clearer.

![image-20250504181148061](./.05_kernel_sched/image-20250504181148061.png)



### struct sched_class

![image-20250504182511093](./.05_kernel_sched/image-20250504182511093.png)

![image-20250504182553430](./.05_kernel_sched/image-20250504182553430.png)



#### **DEFINE_SCHED_CLASS**

/* file: kernel/sched/sched.h */

![image-20250504181727870](./.05_kernel_sched/image-20250504181727870.png)



### stop_sched_class

`/* file: kernel/sched/stop_task.c */`

![image-20250506195257499](./.05_kernel_sched/image-20250506195257499.png)



### dl_sched_class

`/* file: kernel/sched/deadline.c */`

![image-20250504182225095](./.05_kernel_sched/image-20250504182225095.png)



### rt_sched_class

`/* file: kernel/sched/rt.c */`

![image-20250506195226927](./.05_kernel_sched/image-20250506195226927.png)



### fair_sched_class

`/* file: kernel/sched/fair.c */`

![image-20250506194929405](./.05_kernel_sched/image-20250506194929405.png)

![image-20250506194949127](./.05_kernel_sched/image-20250506194949127.png)



#### task_tick_fair

![image-20250509113301083](./.05_kernel_sched/image-20250509113301083.png)



### idle_sched_class

`/* file: kernel/sched.idle.c */`

![image-20250506195144114](./.05_kernel_sched/image-20250506195144114.png)



### ext_sched_class

`/* file: kernel/sched/ext.c */`

![image-20250506194845921](./.05_kernel_sched/image-20250506194845921.png)

![image-20250506194801995](./.05_kernel_sched/image-20250506194801995.png)





## pick_next_task

Implementation of scheduler, which will be called before swapping (/preempting), to choose the next task to switch to.

`/* file: kernel/sched/core.c */`

![image-20250504175723718](./.05_kernel_sched/image-20250504175723718.png)



**for_each_class**

![image-20250504191112142](./.05_kernel_sched/image-20250504191112142.png)

![image-20250504191044561](./.05_kernel_sched/image-20250504191044561.png)



### pick_next_task_fair

![image-20250508120743310](./.05_kernel_sched/image-20250508120743310.png)

![image-20250507182635171](./.05_kernel_sched/image-20250507182635171.png)

![image-20250507182658324](./.05_kernel_sched/image-20250507182658324.png)



#### pick_task_fair

`/* file: kernelj/sched/fair.c */`

![image-20250526220106502](./.05_kernel_sched/image-20250526220106502.png)



##### pick_next_entity

![image-20250525132552505](./.05_kernel_sched/image-20250525132552505.png)



##### pick_eevdf

How the heap and RB-tree are compatible, how is the relationship between `min_vruntime` and `virtual deadline`.

![image-20250526221706432](./.05_kernel_sched/image-20250526221706432.png)

![image-20250530214611655](./.05_kernel_sched/image-20250530214611655.png)



##### entity_eligible

![image-20250601191648089](./.05_kernel_sched/image-20250601191648089.png)

![image-20250601192034350](./.05_kernel_sched/image-20250601192034350.png)



#### put_prev_set_next_task

![image-20250507182753416](./.05_kernel_sched/image-20250507182753416.png)



#### __put_prev_set_next_dl_server

![image-20250507182912536](./.05_kernel_sched/image-20250507182912536.png)



#### put_prev_task_fair

![image-20250507183934752](./.05_kernel_sched/image-20250507183934752.png)



The macro `for_each_sched_entity` is only effective with `CONFIG_FAIR_GROUP_SCHED`.

![image-20250507211639979](./.05_kernel_sched/image-20250507211639979.png)



##### put_prev_entity

![image-20250507184058316](./.05_kernel_sched/image-20250507184058316.png)



#### set_next_task_fair

![image-20250507194438451](./.05_kernel_sched/image-20250507194438451.png)



##### set_next_entity

![image-20250507211433048](./.05_kernel_sched/image-20250507211433048.png)



## struct rq

`rq`是系统可运行任务的容器，调度器的很多工作都是围绕着`rq`来进行的，调度类`struct sched_class`所申明的函数中，绝大多数都与`rq`相关。在系统中，每个CPU都有一个自己的`rq`，这样就可以避免多个CPU访问同一个`rq`产生的并发问题，提升调度器效率。

runqueue: 就续队列

The core of a runqueue, is presumably including:

```c
/* sub-rq for these schedule classes */
struct cfs_rq cfs;
struct rt_rq rt;
struct dl_rq dl;
struct scx_rq scx;

/* Only one task for stop schedule class and idle schedule class */
struct task_struct *idle;
struct task_struct *stop;
```



`/* file: kernel/sched/sched.h */`

![image-20250507221718325](./.05_kernel_sched/image-20250507221718325.png)

![image-20250509114124932](./.05_kernel_sched/image-20250509114124932.png)

![image-20250506154425360](./.05_kernel_sched/image-20250506154425360.png)

![image-20250506154455569](./.05_kernel_sched/image-20250506154455569.png)

![image-20250506154524740](./.05_kernel_sched/image-20250506154524740.png)

![image-20250506154539643](./.05_kernel_sched/image-20250506154539643.png)



### struct dl_rq

![image-20250508112924301](./.05_kernel_sched/image-20250508112924301.png)

![image-20250506160808669](./.05_kernel_sched/image-20250506160808669.png)



### struct rt_rq

![image-20250506162001504](./.05_kernel_sched/image-20250506162001504.png)



#### rt_prio_array

![image-20250506161921855](./.05_kernel_sched/image-20250506161921855.png)



### struct cfs_rq

![image-20250525124236521](./.05_kernel_sched/image-20250525124236521.png)

![image-20250506160420997](./.05_kernel_sched/image-20250506160420997.png)

![image-20250506160449845](./.05_kernel_sched/image-20250506160449845.png)



#### rb_root_cached

带缓存的红黑树只是一个常规的`rb_root`，加上一个额外的指针来缓存最左边的节点。这使得rb_root_cached可以存在于rb_root存在的任何地方，并且只需增加几个接口来支持带缓存的树

`/* file: include/linux/rbtree_types.h */`

![image-20250507213518273](./.05_kernel_sched/image-20250507213518273.png)

![image-20250507213549712](./.05_kernel_sched/image-20250507213549712.png)

![image-20250507213625157](./.05_kernel_sched/image-20250507213625157.png)



### struct scx_rq

![image-20250506160522668](./.05_kernel_sched/image-20250506160522668.png)



## schedule policy



Sched Policy: （调度策略）

- `stop_sched_class`: only one task, no sched policy
- `dl_sched_class`: one policy, `SCHED_DEADLINE`
- `rt_sched_class`: 
  - `SCHED_FIFO`, 一直运行到主动放弃CPU
  - `SCHED_RR` 同优先级任务按照时间配额交替运行
- `fair_sched_class`: 
  - `SCHED_NORMAL`, 用于绝大多数用户进程
  - `SCHED_BATCH`, 用于没有交互行为的后台进程，在完成所有`SCHED_NORAML`的任务后，让该类任务不受打扰地跑上一段时间，以最大限度利用缓存
  - `SCHED_IDLE` 优先级最低任务，只有没有其它任何任务可运行时，调度器才运行该类任务
- `idle_sched_class`: same as `stop`, no sched policy

每个进程在创建时都会指定一个**调度策略**，从而自动归结到某个**调度类**下。



`/* file: include/uapi/linux/sched.h */`

![image-20250504193232984](./.05_kernel_sched/image-20250504193232984.png)



### DL

Deadline 任务有三个重要的属性：runtime, period, deadline，调度器需要确保任务在每个period的时间窗口内得到runtime这么多的CPU时间，并且必须在deadline这个时间点之前得到。

EDF (Earliest Deadline First)，即在任意时刻，调度器都选择Deadline最近的任务执行。

```bash
chrt -d --sched-runtime 500000 --sched-deadline 1000000 \
	--sched-period 1666666 0 video_processing_tool
```



#### CBS

`scheduling_deadline`：子任务的deadline

`remaining_runtime`：在当前period，子任务还剩多少runtime可用



**CBS check**:

`scheduling_deadline < now` check deadline

`remaining_runtime / (scheduling_deadline - now) > runtime / period` check bandwidth



**postpone task** for one period and update states:

`scheduling_deadline = now + deadline`

`remaining_runtime = runtime`



**throttled**:

`remaining_runtime <= 0` will be throttled by scheduler

Then replenished at `scheduling_deadline`, i.e., next `period`. This time is then called `replenishment time`

`scheduling_deadline = scheduling_deadline + period`

`remaining_runtime = remaining_runtime + runtime`



### CFS

Completely Fair Scheduler

CFS吸取了RSDL中关于公平调度的思想，着力于改善调度器在**交互性**与**公平性**两方面的性能。调度器抛弃了基于时间片来划分调度周期的做法，引入了`vruntime`（虚拟时间）的概念来度量公平，并使用**红黒树**来管理任务。

- 现实中绝对公平几乎不可能实现，调度器维持公平性就是去除掉不公平性（维持所有任务的`vruntime`尽可能相等）。
- 公平(fair)调度器的目标是每个进程得到**同样多的CPU时间**，因此调度器每次调度时都选择runtime最小的进程。
- 优先级较高的进程应该得到更多的CPU时间，因此更高的**优先级**（低**nice**值）可以转化为更大的**CPU时间比例**（即时间权重）。
- 权重比例刻画公平性，对比实际使用时间片的比例比与权重是比例的差异来确定不公平性，来调度。
- 虚拟时间（vitrual runtime）是runtime除以**权重**是比例的**归一化数值**，即`vruntime`。
  - `vruntime = (wall_time / (weight / NICE_0_LOAD))`
  - `vruntime = (wall_time * (NICE_0_LOAD * 2^32) / weight)) >> 32 `
  - `vruntime = (wall_time * NICE_0_LOAD * inv_weight) >> 32`
  
- 归一化后，`cfs_rq`中使用`vruntime`作为key值的红黑树来保存所有的任务，最左侧节点的任务就是调度器应该选择的任务。





#### sched_latency

为了**调度延迟**，即一个任务在两次被调度到的时间间隔，CFS也需要引入**调度周期**（`sched_latency`）。

有了调度周期后（`__sched_period()`），还需要为任务计算其在一个调度周期内的时间配额（slice），`sched_slice()`。

Schedule latency is the duration in which all runable processes are expected to be scheduled at least once. It defines the "scheduling period" during which the CPU time is divided fairly among all runable tasks.

Schedule period: effective latency



What is lacking is a way to ensure that some processes can get accesss to a CPU quickly without necessarily giving those processes the ability to obtain more than their fair share of CPU time. (https://lwn.net/Articles/925371/)



##### time slice

time slice for CFS scheduler research

The schedule latency defines the time frame in which all tasks should be sheduled, and the timeslice per task is derived from dividing this latency by the number of tasks, respecting the **minimum granularity**.

This mechanism balances **fairness** (all tasks get CPU time within latency) and **efficiency** (avoid too frequent preemption)



##### minimum granularity

When the number of runnable tasks grows large, the scheduler period (effective latency) increases because each task must get at least the **mininum granularity**. The formula used is:

- If number of runable tasks <= `sched_latency_ns / sched_min_granularity_ns`, then scheduling period = `sched_latency_ns`
- Else, scheduling period = number of runnable tasks `x` `sched_min_granularity_ns`

This ensures that each task gets a fair minimum CPU slice, preventing too frequent context switches.



##### Handling New/Waking Tasks

This is **critial** as it's normal for tasks to go to sleep or wait for some resources and then they will wake up later. Newly created tasks usually expect to execute quickly to be responsive, but it shouldn't occupy CPU too long to starve other runnables.



#### EEVDF scheduler

Earliest Eligible **Virtual Deadline** First, A newer scheduler variant alongside CFS

The `commit` adding this implementation is `git show 147f3efaa241`



EEVDF picks task with `lag` greater or equal to zero and calculates a virtual deadline (**VD**) for each, selecting the task with the earliest VD to execute next.

It aims to improve fairness, **responsiveness**, and scalability while addressing **limitations** in CFS.

Tick driven preemption is driven by `request/slice` completion; 

while wakeup preemption is driven by the `deadline`.



**virtual `deadline`**

The other factor that comes into play is the "virtual deadline", which is the ~~earliest~~ time by which a process should have received its due CPU time. This deadline is calculated by adding a process's allocated time slice to its **eligible time**. A process with a 10ms time slice, and whose eligible time is 20ms in the future, will have a virtual deadline that is 30ms in the future.

A task is deemed "**eligible**" if its lag value is zero or greater; whenever the CPU scheduler must pick a task to run, it chooses from the set of eligible tasks. **eligible time** is the virtual **runtime of an eligible task**.

`deadline = vruntime + (slice / weight) * total_weight`  ?

Deadline is the time when a process should have received its due CPU timeslice, while all other processess get their partitions too. (This is why it multiplies the **total_weight** after being normalized)





**virtual `lag`**

Imagine a time period of one second; during that time, in our five-process scenario, each process should have gotten 200ms of CPU time. For a number of reasons, things never turn out exactly that way; some processes will have gotten too much time, while others will have been shortchanged. For each process, EEVDF calculates the difference between the time that process **should** have gotten and how much it **actually** got; that difference is called "lag". A process with a positive lag value has not received its fair share and should be scheduled sooner than one with a negative lag value.

The `lag` value is calculated over the normalized virtual runtime, since its easier to get the target virtual runtime (`rq.avg_vruntime`) for tasks in `runqueue`:

`se.vlag = rq.avg_vruntime - se.vruntime`



**Normalized** virtual runtime is the key dimension in CFS scheduler:

`se.vruntime = runtime / weight`

`se.vruntime += timeslice / weight`

`weight = load_weight.weight / NICE_0_LOAD`

`weight` is the **weight** of an entity. It is used to normalize the `runtime` from entities with different time weights. (`NICE_0_LOAD` is the weight of a default-priority task)



`runtime`s are supposed to be **evenly** separated to all entities in `runqueue`, so the `rq.avg_vruntime` means the **normalized** virtual runtime should be entitled to each task:

`rq.avg_vruntime = total_runtimes / total_weights`

`total_weights` is **sum of weights** of all runnable tasks in the runqueue.

`total_runtimes` is **sum of runtimes** the runqueue having gone through.



The sum of all the `lag` values in the system is always zero.

`lag = se.vlag * (weight)`

`weight` is the **weight** of an entity.

```
0 = sum of lags 
= sum of ((rq.avg_vruntime - se.vruntime) * weight)s
= sum of ((total_runtimes / total_weights - runtime / weight) * weight)s
= sum of ((total_runtimes * weight / total_weights - runtime)s
= sum of (total_runtimes * weight / total_weights)s - sum of runtimes
= total_runtimes * (sum of (weight / total_weights)s - 1)
= total_runtimes * (sum of (weight)s / total_weights - 1)

sum of (weight)s = sum of weights in runqueue = total_weights
```



```
0 = sum of vlags ?
= sum of (rq.avg_vruntime - se.vruntime)s
= sum of ((total_runtimes) / (total_weights) - runtime / weight)s
= n * (total_runtimes) / (total_weights) - sum of (runtime / weight)s
```







a scheduler includes: base scheduler, placement, preemption, picking



##### limitations in CFS

To figure out the reason why EEVDF was involved in CFS!

Corner cases: task groups starving under load

heterogeneous workloads: mixed CPU-bound/I/O-bounds tasks

low-latency scheduling for time-sensitive applications





##### sched_setattr

`/* file: kernel/sched/syscalls.c */`

![image-20250517120551669](./.05_kernel_sched/image-20250517120551669.png)



##### __setparam_fair

`/* file: kernel/sched/fair.c */`

![image-20250517120151062](./.05_kernel_sched/image-20250517120151062.png)





##### sysctl_sched_base_slice

`/* file: kernel/sched/fair.c */`

![image-20250519222613136](./.05_kernel_sched/image-20250519222613136.png)



![image-20250520144002638](./.05_kernel_sched/image-20250520144002638.png)



![image-20250520120324883](./.05_kernel_sched/image-20250520120324883.png)

![image-20250520120412672](./.05_kernel_sched/image-20250520120412672.png)



![image-20250520122536609](./.05_kernel_sched/image-20250520122536609.png)

![image-20250520122156843](./.05_kernel_sched/image-20250520122156843.png)



##### sched_init_debug

`/* file: kernel/sched/debug.c */`

![image-20250520212340971](./.05_kernel_sched/image-20250520212340971.png)

![image-20250520211705176](./.05_kernel_sched/image-20250520211705176.png)

![image-20250520211802700](./.05_kernel_sched/image-20250520211802700.png)



##### base_slice_ns

![image-20250520212228241](./.05_kernel_sched/image-20250520212228241.png)







#### struct load_weight

![image-20250508151508018](./.05_kernel_sched/image-20250508151508018.png)

##### sched_prio_to_weight

![image-20250508150808509](./.05_kernel_sched/image-20250508150808509.png)

##### sched_prio_to_wmult

![image-20250508150158944](./.05_kernel_sched/image-20250508150158944.png)



#### update_curr

`/* file: kernel/sched/fair.c */`

![image-20250521163356569](./.05_kernel_sched/image-20250521163356569.png)

![image-20250508175544523](./.05_kernel_sched/image-20250508175544523.png)



##### update_curr_se

![image-20250508193522518](./.05_kernel_sched/image-20250508193522518.png)



##### update_curr_task

![image-20250508193617042](./.05_kernel_sched/image-20250508193617042.png)



##### update_deadline

![image-20250521163321601](./.05_kernel_sched/image-20250521163321601.png)



###### calc_delta_fair

![image-20250508175730771](./.05_kernel_sched/image-20250508175730771.png)

![image-20250601231804834](./.05_kernel_sched/image-20250601231804834.png)



##### update_min_vruntime

`min_vruntime` is used to calculate increment of `avg_vruntime`, which is then used for `lag` caculation.

![image-20250601224756223](./.05_kernel_sched/image-20250601224756223.png)



##### __update_min_vruntime

![image-20250521163807023](./.05_kernel_sched/image-20250521163807023.png)



###### avg_vruntime_update

![image-20250525101235538](./.05_kernel_sched/image-20250525101235538.png)



###### __pick_root_entity

![image-20250525125028422](./.05_kernel_sched/image-20250525125028422.png)



#### sched_tick

调度节拍

`update_process_times() -> sched_tick() -> task_tick() -> entity_tick()`



##### entity_tick

`/* file: kernel/sched/fair.c */`

![image-20250508160612911](./.05_kernel_sched/image-20250508160612911.png)



##### resched_curr_lazy

`/* file: kernel/sched/core.c */`

![image-20250508161052609](./.05_kernel_sched/image-20250508161052609.png)

![image-20250508161543459](./.05_kernel_sched/image-20250508161543459.png)



##### __resched_curr

![image-20250508161923002](./.05_kernel_sched/image-20250508161923002.png)



# Add task



## sched_fork

![image-20250520145535737](./.05_kernel_sched/image-20250520145535737.png)

![image-20250520145348299](./.05_kernel_sched/image-20250520145348299.png)



### __sched_fork

![image-20250520145502064](./.05_kernel_sched/image-20250520145502064.png)



## queue_task_fair

`/* file: kernel/sched/fair.c */`

![image-20250517115650956](./.05_kernel_sched/image-20250517115650956.png)

![image-20250517112303017](./.05_kernel_sched/image-20250517112303017.png)

![image-20250517112446884](./.05_kernel_sched/image-20250517112446884.png)

![image-20250517112513540](./.05_kernel_sched/image-20250517112513540.png)



### cfs_rq_min_slice

![image-20250517115113947](./.05_kernel_sched/image-20250517115113947.png)



## enqueue_entity

`/* file: kernel/sched/fair.c */`

![image-20250509180658352](./.05_kernel_sched/image-20250509180658352.png)

![image-20250509180810126](./.05_kernel_sched/image-20250509180810126.png)



### place_entity

![image-20250601235819940](./.05_kernel_sched/image-20250601235819940.png)

![image-20250601235629475](./.05_kernel_sched/image-20250601235629475.png)

![image-20250601235901442](./.05_kernel_sched/image-20250601235901442.png)



### __enqueue_entity

`/* file: kernel/sched/fair.c */`

![image-20250519212206205](./.05_kernel_sched/image-20250519212206205.png)



# sched_init

`/* file: kernel/sched/core.c */`

![image-20250513224446082](./.05_kernel_sched/image-20250513224446082.png)

![image-20250513224558687](./.05_kernel_sched/image-20250513224558687.png)

![image-20250513224623512](./.05_kernel_sched/image-20250513224623512.png)

![image-20250513224650721](./.05_kernel_sched/image-20250513224650721.png)

![image-20250519222437726](./.05_kernel_sched/image-20250519222437726.png)

![image-20250513224818789](./.05_kernel_sched/image-20250513224818789.png)



# task_struct

`/* file: include/linux/sched.h */`

![image-20250429160235585](./.05_kernel_sched/image-20250429160235585.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_3c21493c8ac91bca.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_51d5f1efe6470f1d.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_6f6af38229846a07.png)

![image-20250429182555223](./.05_kernel_sched/image-20250429182555223.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_9de8e0f00533db5a.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_a52e80cdbeec0149.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_8ecfb61d6d810625.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_d2849e7d79ecd0fb.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_5e8f500418d4fb84.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_aef7d0888376d13a.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_fbacce374ff5e798.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_d197c7acbfe88a2c.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_77f7b5120eecc5e9.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_2602a02fb240e6d7.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_8636bfa1bb565e0a.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_ef1357ce15a68d85.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_436ab371f7b02e95.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_b8689bf1978f75bc.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_ccc7f28fbd5652bd.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_c3baab03f8fb9da8.png)

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_5cb7644c8a51fff3.png)

![image-20250429160159489](./.05_kernel_sched/image-20250429160159489.png)



## new

Implementation in newer version code (v6.14-rc5):

![image-20250506164922578](./.05_kernel_sched/image-20250506164922578.png)

![image-20250506163720063](./.05_kernel_sched/image-20250506163720063.png)

![image-20250507220315485](./.05_kernel_sched/image-20250507220315485.png)

![image-20250506163844565](./.05_kernel_sched/image-20250506163844565.png)

![image-20250506163917020](./.05_kernel_sched/image-20250506163917020.png)

![image-20250506163950335](./.05_kernel_sched/image-20250506163950335.png)

![image-20250506164021383](./.05_kernel_sched/image-20250506164021383.png)

![image-20250506164048287](./.05_kernel_sched/image-20250506164048287.png)

![image-20250506164127481](./.05_kernel_sched/image-20250506164127481.png)

![image-20250506164155057](./.05_kernel_sched/image-20250506164155057.png)

![image-20250506164234430](./.05_kernel_sched/image-20250506164234430.png)

![image-20250506164317639](./.05_kernel_sched/image-20250506164317639.png)

![image-20250506164351835](./.05_kernel_sched/image-20250506164351835.png)

![image-20250506164425294](./.05_kernel_sched/image-20250506164425294.png)

![image-20250506164450171](./.05_kernel_sched/image-20250506164450171.png)

![image-20250506164511871](./.05_kernel_sched/image-20250506164511871.png)

![image-20250506164536498](./.05_kernel_sched/image-20250506164536498.png)

![image-20250506164601256](./.05_kernel_sched/image-20250506164601256.png)

![image-20250506164623160](./.05_kernel_sched/image-20250506164623160.png)

![image-20250506164649105](./.05_kernel_sched/image-20250506164649105.png)

![image-20250506164717901](./.05_kernel_sched/image-20250506164717901.png)

![image-20250506164755251](./.05_kernel_sched/image-20250506164755251.png)



## thread_info

/* file: arch/riscv/include/asm/thread_info.h */

![image-20250429161736647](./.05_kernel_sched/image-20250429161736647.png)



## thread_struct

.thread在`task_struct`结构体的最后边。

线程上下文（vs pt_regs），线程换出时保存，换入时恢复。

![img](./.05_kernel_sched/lu1676272gu7a4z_tmp_fd521063a6463d03.png)



## kernel stack

For **PID0** process, the kernel stack is on a thread block together with`task_struct` `init_task` . The total size of the thread block is 16K. But for other threads, their **kernel stack** and `task_struct` will allocated separately.

/* file: arch/riscv/include/asm/processor.h */

![image-20250429160505720](./.05_kernel_sched/image-20250429160505720.png)

/* file: include/asm-generic/vmlinux.lds.h */

![image-20250429160543713](./.05_kernel_sched/image-20250429160543713.png)



### THREAD_SIZE

The thread block size is 16K (4 page size) on 64-bit system, 8K on 32-bit system.

/* file: arch/riscv/include/asm/thread_info.h */

![image-20250429161040849](./.05_kernel_sched/image-20250429161040849.png)





# sched_tick

调度节拍

`/* file: kernel/sched/core.c */`

![image-20250508171828351](./.05_kernel_sched/image-20250508171828351.png)

![image-20250508171849191](./.05_kernel_sched/image-20250508171849191.png)



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

`/* file: include/linux/hrtimer_types.h */`

![image-20250508112254176](./.05_kernel_sched/image-20250508112254176.png)



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

**Is the number of system ticks**. The system tick frequency is set by `CONFIG_HZ`, while 100 means ticking 100 times in a second. The tick period is 10ms for `CONFIG_HZ=100`, as `tick = 1s / HZ`.

