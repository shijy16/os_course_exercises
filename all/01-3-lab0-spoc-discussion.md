# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  - 硬件上直接的支持
    - 进程：硬件支持时钟中断
    - 虚存：地址映射需要MMU等硬件
    - 文件系统：需要稳定的存储介质
  - 特权指令
    - 进程：中断使能，软中断触发等中断相关指令
    - 虚存：内存寻址模式切换，设置页表等内存相关指令
    - 文件系统：I/O操作等文件操作相关指令

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  - 区别：

    - 实模式：将物理内存看作分段的区域，程序和代码分开存放，进程直接使用物理地址访问内存，可以修改系统程序或者用户程序代码。
    - 保护模式：用户程序使用虚拟地址访问内存，由操作系统进行地址映射，应用程序不能直接访问物理内存地址。

  - 含义

    - 物理地址：处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。
    - 逻辑地址：在有地址变换功能的计算机中，内存访问指令给出的地址。
    - 线性地址：逻辑地址和物理地址变换的中间层，线性地址空间是操作系统的虚存管理下每个应用程序能够访问的地址空间。可以让多个运行的应用程序相互隔离。

    逻辑地址通过段机制映射为线性地址，线性地址通过页机制映射为物理地址。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

  - 区别
    - 用户模式：为不受信任的进程提供了隔离保护，禁止不可新的代码执行特权指令和访问特权控制状态指令，不可信的进程只能访问自己那部分内存。一旦发生中断和异常，均切换到机器模式进行处理。
    - 机器模式：拥有最高权限，对底层功能有完全的使用权，可以不受限制地访问整个机器。是所有riscv架构必须具备的模式。
    - 监督模式：使用了页机制的虚拟内存，权限比机器模式低，比用户模式高。部分异常处理不需要切换到机器模式。
  - 地址访问特征：
    - 机器模式：直接使用物理地址访问内存。
    - 用户模式：通过物理内存保护功能（PMP）限制进程访存和配置寄存器，当进程访存时，会将地址和PMP地址寄存器比较，决定是否可以继续访存。
    - 监督模式：使用页机制进行虚拟地址映射，将虚拟地址转换为物理地址。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义

```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

表示该域在结构体中所占位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

intr仅32位，计算得0x20003（小端？）

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

````assembly
    7c5c:	ff                   	(bad)  
    7c5d:	ff 00                	incl   (%eax) #%eax++
    7c5f:	00 00                	add    %al,(%eax) #%al，%eax求和后放入%eax
    7c61:	9a cf 00 ff ff 00 00 	lcall  $0x0,$0xffff00cf #过程调用，调用对应地址处函数
    7c68:	00                   	.byte 0x0               #nop?
    7c69:	92                   	xchg   %eax,%edx	#交换%eax和%edx的值
    7c6a:	cf                   	iret 				#返回
````



1. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

+ 特定数据值的访问

  + 如`#define IRQ_OFFSET      32`

+ 类型转换

  + 如`#define to_struct(ptr, type, member)  ((type *)((char *)(ptr) - offsetof(type, member)))`

+ 常用代码片段复用

  + ````
    #define do_div(n, base) ({                                        \
        unsigned long __upper, __low, __high, __mod, __base;        \
        __base = (base);                                            \
        asm("" : "=a" (__low), "=d" (__high) : "A" (n));            \
        __upper = __high;                                            \
        if (__high != 0) {                                            \
            __upper = __high % __base;                                \
            __high = __high / __base;                                \
        }                                                            \
        asm("divl %2" : "=a" (__low), "=d" (__mod)                    \
            : "rm" (__base), "0" (__low), "1" (__upper));            \
        asm("" : "=A" (n) : "a" (__low), "d" (__high));                \
        __mod;                                                        \
     })
    ````



#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
