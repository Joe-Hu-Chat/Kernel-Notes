2023年8月28日星期一



# Trace function in Linux kernel



## ftrace

ftrace framework:internal tracers for kernel tracing

tracepoints: predefined trace locations

event hooks: custom trace events at specific places in the code

tracers: modules within the ftrace framework that collect and process data



BPF(Berkeley Packet Filter): bpftrace: trace user-space programs



使用方法：

### 内核配置剪裁

```
make ARCH=riscv menuconfig egret_defconfig
```

使能 ->Kernel hacking ->Tracers ->Interrupts-off Latency Tracer

![img](./.22_trace/lu1279811hw7em_tmp_1f9debdd1649b73.png)



### 安装tracefs

/etc/fstab:

```
tracefs /sys/kernel/tracing tracefs defaults 0 0
```



or mount at run time:

```
mount -t tracefs nodev /sys/kernel/tracing
```



legacy in debugfs

当安装debugfs文件系统后，`/sys/kernel/debug/tracing`中的内容和`tracefs`中的内容一样。



### 设置tracer



`current_tracer`:

可以用来显示或者配置当前配置的tracer。

![img](./.22_trace/lu1279811hw7em_tmp_7ee0b0bef9fe0956.png)



`available_tracers`:

编译进内核的可以用的，可以写进`current_tracer`的tracer。

![img](./.22_trace/lu1279811hw7em_tmp_61c7f48bb1b2b4b9.png)



将`irqsoff`写入`current_tracer`:

![img](./.22_trace/lu1279811hw7em_tmp_1b43fc94ab0ae6ac.png)



### 开启和关闭tracing

开启：`echo 1 > tracing_on`

关闭：`echo 0 > tracing_on`



### trace | trace_pipe

`trace`: 非消耗性（not a consumer）输出

![img](./.22_trace/lu1279811hw7em_tmp_ca6a16036f535053.png)



`trace_pipe`: live tracing，读消耗性（a consumer），会等待新数据



### events

![img](./.22_trace/lu1279811hw7em_tmp_7d9b2d4270851660.png)

![img](./.22_trace/lu1279811hw7em_tmp_ce902da11898994d.png)

![img](./.22_trace/lu1279811hw7em_tmp_f09f0a9b740dc632.png)

使能相应的事件：irq

![img](./.22_trace/lu1279811hw7em_tmp_6bed54f4b674304a.png)

### 停止tracing

```
echo 0 > tracing_on
echo nop > current_tracer
umount /sys/kernel/tracing
```



## Trace irqflags

### `irqs_disabled()`

![img](./.22_trace/lu1279811hw7em_tmp_d7683d4ae3bc09a2.png)

![img](./.22_trace/lu1279811hw7em_tmp_13cf1250fc3fab37.png)

![img](./.22_trace/lu1279811hw7em_tmp_c817bd2779494d83.png)

![img](./.22_trace/lu1279811hw7em_tmp_288a5c76e859485c.png)

![img](./.22_trace/lu1279811hw7em_tmp_dee6bb6514341bc.png)



![img](./.22_trace/lu1279811hw7em_tmp_1784addeaf686c3e.png)

![img](./.22_trace/lu1279811hw7em_tmp_b86c47ced08c4f26.png)



![img](./.22_trace/lu1279811hw7em_tmp_8591c18ba8291a1c.png)



![img](./.22_trace/lu1279811hw7em_tmp_4e04a46583a15e63.png)



### `CONFIG_TRACE_IRQFLAGS`

在中断例程中，有以下跟irqflags相关的trace操作：

1. ​	进中断时，保存寄存器现场后，记录中断关闭

![img](./.22_trace/lu1279811hw7em_tmp_dc1f1f119502cc35.png)

1. ​	在处理异常的情况下，会再记录中断开启的操作（异常响应时中断是根据SPIE再打开的，而中断响应时全局中断被硬件关闭后未再由软件打开，但SPIE应该是1才能使中断返回时硬件开启全局中断）

![img](./.22_trace/lu1279811hw7em_tmp_9cca4e15f4e8445e.png)

1. ​	异常处理结束后，会再关闭中断（因为sret时，硬件会恢复中断现场，其中也包括全局中断位），并记录操作

![img](./.22_trace/lu1279811hw7em_tmp_60e2d5712cc8bfd6.png)

1. ​	中断响应结束前会恢复寄存器现场，在这之前会根据`SPIE`的值，测定sret是会关闭中断，还是开启中断，并提前记录相应的操作

![img](./.22_trace/lu1279811hw7em_tmp_a85eb579a070f028.png)



### trace_hardirqs_on/off



![img](./.22_trace/lu1279811hw7em_tmp_b18db295725f2101.png)

内核配置irqflags后，就会具备相应的trace功能。

trace操作只有在第一次开启中断的时候才会记录（由tracing_irq_cpu来记录是否是，关闭后第一次打开）

![img](./.22_trace/lu1279811hw7em_tmp_6e8b1cf46f188326.png)

trace操作只有在第一次关闭中断的时候才记录（由tracing_irq_cpu来跟踪是否是，打开后第一次关闭）。



![img](./.22_trace/lu1279811hw7em_tmp_e2d72a367195704f.png)

每次执行`trace_hardirqs_on`时都会把置1的`tracing_irq_cpu`清零；每次执行`trace_hardirqs_off`时都会把清零的`tracing_irq_cpu`置1。这样就可以记录下中断持续处于关闭状态的时间。





关闭中断时，开始计时（会受到irq trace和preempt trace配置的影响）

![img](./.22_trace/lu1279811hw7em_tmp_8c98d8e7e6f7fc1f.png)

打开中断时，结束计时，这样就得出全局中断关闭的时间。

![img](./.22_trace/lu1279811hw7em_tmp_dab0ee8480f208e9.png)



只有相应的tracer开启了，且全局中断处于关闭状态才启动和停止精确计时。

![img](./.22_trace/lu1279811hw7em_tmp_47ca890e87245dc4.png)

如果preempt tracer也配置了，且开启了，则会根据preemption来决定要不要启动或者停止计时。只有在preemption使能的时候才启动或者停止计时。

![img](./.22_trace/lu1279811hw7em_tmp_f1173523af109d4d.png)



如果没有配置`CONFIG_TRACE_IRQFLAGS`，则相应的trace操作的宏相当于空

![img](./.22_trace/lu1279811hw7em_tmp_26bf35c532a8ce9f.png)



### lockdep

用来检测潜在的deadlock或者其它的同步问题。



![img](./.22_trace/lu1279811hw7em_tmp_3ca184c1a3e4efba.png)