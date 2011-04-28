# linux内核延时操作 #

系统为了实现中断的快速处理，将中断处理程序分为处理中断请求的上半部和延时处理逻辑和I/O的下半部（buttom halves）。下半部的主要实现机制有两种：tasklet和workqueue。

## tasklets ##

tasklet是软件中断（softirq）机制的一部分，tasklet运行在软件中断上下文中，所以有软件中断服务函数的所有要求，比如不能休眠。tasklet相对于软件中断有一些针对驱动编写的好处：

- 通过软件中断实现，回调函数不能睡眠
- tasklet的分配是动态的，可以在内核运行时添加和删除
- 同类型的tasklet总是串行执行的，因为它的主要数据结构是per-cpu的，所以不需要人为使用spinlock。但是不同的tasklet则可以并发地在不同cpu上运行。
- tasklet的串行化使tasklet函数不必是可重入的。
- 不能访问用户态进程地址

### softirq implementation  ###

`kernel/softirq.c`

- `static __init int spawn_ksoftirqd(void);` 创建softirq的内核线程，用于延迟处理过多的softirq，这是softirq处理速度和cpu公平性的一个折中处理。
- `kthread_create(run_ksoftirqd, hcpu, "ksoftirqd/%d", hotcpu);` 设置内核线程的回调函数`run_ksoftirqd`
- `static int run_ksoftirqd(void * __bind_cpu);` 内核线程回调，`if (local_softirq_pending()) __do_softirq();`如果本地cpu有待处理的softirq，则调用softirq的处理函数
- `asmlinkage void __do_softirq(void);` 根据`softirq_vec`对应中对应的位来调用相应的`action`，循环处理所有待处理的softirq，处理过程中间如果有新来的softirq则会重新处理max_restart这么多次。如果处理max_restart次之后还有待处理的softirq则会重新调度ksoftirqd继续处理。

### tasklet implementation ###

- `void __init softirq_init(void);`初始化softirq， 调用`open_softirq(TASKLET_SOFTIRQ, tasklet_action);` 将tasklet的回调函数`tasklet_action`注册到softirq的处理函数中。这样在有tasklet对应的softirq处理时就会调用`tasklet_action`。

- `static void tasklet_action(struct softirq_action *a);`这个函数主要有两个功能：
 - `t->func(t->data);`如果tasklet需要被调度，则运行它的回调函数
 - `tasklet_trylock(t);`测试tasklet是否在其他cpu上运行， `test_and_set_bit(TASKLET_STATE_RUN, &(t)->state);` 如果正在其他cpu上运行则重新发softirq请求延迟执行， `__raise_softirq_irqoff(TASKLET_SOFTIRQ);`

## workqueue ##

`kernel/workqueue.c`

工作队列的具体实现比较复杂，但是它的原理很简单，就是创建内核线程来执行函数。

### 主要的实现函数 ###

- `static void process_one_work(struct worker *worker, struct work_struct *work)` 在worker线程上调用`struct work_struct`的回调函数。

### 主要的接口 ###

请参考`linux/workqueue.h`

### 主要特点 ###

- 用内核线程实现，运行在进程上下文，回调函数可以睡眠
- 在一个cpu上创建的work可能在其他cpu上运行
- 不能访问进程的用户态地址，因为这些内核线程没有相应的用户线程
- 可以指定延迟一段具体的时间执行

## resources ##

- [Kernel Hacking Guide](http://www.kernel.org/doc/htmldocs/kernel-hacking.html)
- [workqueue.txt](http://www.mjmwired.net/kernel/Documentation/workqueue.txt)
- [Deferrable functions by Tim](http://www.ibm.com/developerworks/linux/library/l-tasklets/)
