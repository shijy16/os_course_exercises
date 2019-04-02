#lec11: 进程／线程概念spoc练习

## 视频相关思考题

### 11.1 进程的概念

1. 什么是程序？什么是进程？
   1. 程序是对实现预期目的的一系列动作执行过程的描述，由一系列操作指令序列组成
   2. 进程是具有一定独立功能的程序在一个数据集合上一次动态执行的过程


2. 进程有哪些组成部分？

   1. 程序、数据、执行状态


3. 请举例说明进程的独立性和制约性的含义。

   1. 独立性：进程是资源分配和调度的基本单位
   2. 制约性：进程之间有共享资源或者通信


4. 程序和进程联系和区别是什么？

   1. 联系：程序是进程的一部分，同一程序可以有多个不同进程
   2. 区别：进程是动态的执行状态，程序是静态的指令序列。进程是暂时的，程序是永久的


### 11.2 进程控制块

1. 进程控制块的功能是什么？
   1. 管理和控制进程运行


2. 进程控制块中包括什么信息？

   1. 进程基本信息：进程标识PID
   2. 进程控制信息：调度、通信和资源占用
   3. 运行状态：处理器现场保存


3. ucore的进展控制块数据结构定义中哪些字段？有什么作用？


   1. ````c
      struct proc_struct {
          enum proc_state state;                      // Process state
          int pid;                                    // Process ID
          int runs;                                   // the running times of Proces
          uintptr_t kstack;                           // Process kernel stack
          volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
          struct proc_struct *parent;                 // the parent process
          struct mm_struct *mm;                       // Process's memory management field
          struct context context;                     // Switch here to run process
          struct trapframe *tf;                       // Trap frame for current interrupt
          uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
          uint32_t flags;                             // Process flag
          char name[PROC_NAME_LEN + 1];               // Process name
          list_entry_t list_link;                     // Process link list 
          list_entry_t hash_link;                     // Process hash list
          int exit_code;                              // exit code (be sent to parent proc)
          uint32_t wait_state;                        // waiting state
          struct proc_struct *cptr, *yptr, *optr;     // relations between processes
          struct run_queue *rq;                       // running queue contains Process
          list_entry_t run_link;                      // the entry linked in run queue
          int time_slice;                             // time slice for occupying the CPU
          skew_heap_entry_t lab6_run_pool;            // FOR LAB6 ONLY: the entry in the run pool
          uint32_t lab6_stride;                       // FOR LAB6 ONLY: the current stride of the process 
          uint32_t lab6_priority;                     // FOR LAB6 ONLY: the priority of process, set by lab6_set_priority(uint32_t)
      };
      ````

      


### 11.3 进程状态

1. 进程生命周期中的相关事件有些什么？它们对应的进程状态变化是什么？
   1. 接收：创建->就绪挂起/就绪
   2. 激活：进程从挂起状态到等待/就绪状态，即从硬盘中移动到内存中。
   3. 挂起：进程从运行/等待/就绪到等待挂起/就绪挂起，即从内存中移动到了硬盘中。
   4. 事件发生：从等待(挂起)到就绪(挂起)
   5. 等待事件：从运行到等待。
   6. 调度：从就绪到运行。
   7. 时间片完：从运行到就绪。
   8. 结束：从运行到退出


### 11.4 三状态进程模型

1. 运行、就绪和等待三种状态的含义？7个状态转换事件的触发条件是什么？
   1. 运行：进程占用CPU执行
   2. 就绪：进程在就绪队列中等待CPU资源，已经获取了其他资源
   3. 等待：等待某事件，暂停执行
   4. 七个转换事件：
      1. 启动：进程被创建
      2. 进入就绪队列：进程获取到其他资源/等待到了事件
      3. 被调度：就绪队列前面进程处理完毕，进程占用CPU开始执行
      4. 等待事件：进程需要某操作系统提供某资源，需要等待事件发生。
      5. 时间片用完：时钟中断发生
      6. 事件发生：操作系统准备好了进程需要资源，发送通信信号
      7. 结束：进程出错退出/正常退出


### 11.5 挂起进程模型

1. 引入挂起状态的目的是什么？
   1. 减少进程占用内存，把进程控制信息映射到外存
2. 引入挂起状态后，状态转换事件和触发条件有什么变化？
   1. 就绪被分为就绪和就绪挂起
   2. 等待被分为等待和等待挂起
   3. 触发条件：多了激活和挂起
3. 内存中的什么内容放到外存中，就算是挂起状态？
   1. 进程内核栈被放入外存。
      1. 进程内核栈是进程陷入内核态时的运行栈，进程再用户态运行时总是为空

### 11.6 线程的概念

1. 引入线程的目的是什么？
   1. 在同一地址空间实现并发函数执行，共享资源
2. 什么是线程？
   1. 是进程中描述指令流执行状态的组成部分，是CPU调度的基本单位
3. 进程与线程的联系和区别是什么？
   1. 联系：线程是进程的组成部分
   2. 进程是资源分配基本单位：进程地址空间，进程占用资源
   3. 线程是CPU调度基本单位：CPU寄存器信息，堆栈信息
   4. 进程内多个线程共享相同进程地址空间

### 11.7 用户线程

1. 什么是用户线程？
   1. 以**用户函数库**形式提供的线程实现机制
2. 用户线程的线程控制块保存在用户地址空间还是在内核地址空间？
   1. 用户地址空间


### 11.8 内核线程

1. 用户线程与内核线程的区别是什么？

   1. 用户线程是用户函数库在用户态实现的线程机制，不需要内核支持

   2. 内核线程是内核通过系统调用实现的线程机制

   3. 区别：实现方式、TCB位置(内核空间内为每一个内核支持线程设置了一个线程控制块（TCB），内核根据该控制块，感知线程的存在，并进行控制)、运行开销、线程阻塞影响的范围

   4. > > reference
      > >
      > >  用户线程指不需要内核支持而在用户程序中实现的线程，其不依赖于操作系统核心，应用进程利用线程库提供创建、同步、调度和管理线程的函数来控制用户线程。不需要用户态/核心态切换，速度快，操作系统内核不知道多线程的存在，因此一个线程阻塞将使得整个进程（包括它的所有线程）阻塞。由于这里的处理器时间片分配是以进程为基本单位，所以每个线程执行的时间相对减少。
      > >
      > > **以下是用户级线程和内核级线程的区别：**
      > >
      > > **（1）**内核支持线程是OS内核可感知的，而用户级线程是OS内核不可感知的。
      > >
      > > **（2）**用户级线程的创建、撤消和调度不需要OS内核的支持，是在语言（如Java）这一级处理的；而内核支持线程的创建、撤消和调度都需OS内核提供支持，而且与进程的创建、撤消和调度大体是相同的。
      > >
      > > **（3）**用户级线程执行系统调用指令时将导致其所属进程被中断，而内核支持线程执行系统调用指令时，只导致该线程被中断。
      > >
      > > **（4）**在只有用户级线程的系统内，CPU调度还是以进程为单位，处于运行状态的进程中的多个线程，由用户程序控制线程的轮换运行；在有内核支持线程的系统内，CPU调度则以线程为单位，由OS的线程调度程序负责线程的调度。
      > >
      > > **（5）**用户级线程的程序实体是运行在用户态下的程序，而内核支持线程的程序实体则是可以运行在任何状态下的程序。

2. 同一进程内的不同线程可以共用一个相同的内核栈吗？

   1. 用户线程可以

3. 同一进程内的不同线程可以共用一个相同的用户栈吗？

   1. 不可以


## 选做题
1. 请尝试描述用户线程堆栈的可能维护方法。

## 小组思考题
(1) 熟悉和理解下面的简化进程管理系统中的进程状态变化情况。
 - [简化的三状态进程管理子系统使用帮助](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.md)
 - [简化的三状态进程管理子系统实现脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.py)

(2) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程。在理解[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)的基础上，完成＂YOUR CODE"部分的内容。然后通过测试用例和比较自己的实现与往届同学的结果，评价自己的实现是否正确。可２个人一组。

### 进程的状态 

 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - DONE    - 进程结束

### 进程的行为
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU


### 进程调度
 - 使用FIFO/FCFS：先来先服务,
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc 
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数; 
   Ｙ是执行CPU的比例(0..100) ，如果是100，表示不会发出yield操作
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例

#### 例１
```
$./process-simulation.py -l 5:50
Process 0
  yld
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0 
  1     RUN:yld 
  2     RUN:yld 
  3     RUN:cpu 
  4     RUN:cpu 
  5     RUN:yld 

```


#### 例２
```
$./process-simulation.py  -l 5:50,5:50
Produce a trace of what would happen when you run these processes:
Process 0
  yld
  yld
  cpu
  cpu
  yld

Process 1
  cpu
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0     PID: 1 
  1     RUN:yld      READY 
  2       READY    RUN:cpu 
  3       READY    RUN:yld 
  4     RUN:yld      READY 
  5       READY    RUN:cpu 
  6       READY    RUN:cpu 
  7       READY    RUN:yld 
  8     RUN:cpu      READY 
  9     RUN:cpu      READY 
 10     RUN:yld      READY 
 11     RUNNING       DONE 
```
