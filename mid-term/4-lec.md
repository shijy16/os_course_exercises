# lec11 进程/线程

## 进程

> 进程切换会切换栈，不会切换页表

进程控制块PCB，是进程存在的唯一标志

每个进程再操作系统中都有对应PCB

是资源分配的基本单位

##### PCB包含

+ PC
+ SP
+ 其他寄存器
+ PID
+ UID
+ 调度优先级
+ 打开文件列表

进程状态：创建、运行、等待、就绪、退出

## 线程

是进程的一部分，是CPU调度的基本单位，是进程中指令执行流的最小单元

![1554642779969](D:\code\os\os_course_exercises\mid-term\pic\1554642779969.png)

![1554642919386](D:\code\os\os_course_exercises\mid-term\pic\1554642919386.png)

# lec12

## fork()

创建子进程

## exec()

加载新程序取代当前进程（覆盖后续代码）

## wait()

子进程结束时通过exit向父进程返回一个值

父进程通过wait接受并处理

+ 有子进程存活时，父进程等待子进程返回结果
+ 有僵尸子进程等待时，wait()立即返回其中一个值
+ 无子进程存活时，wait()立即返回

## exit()

将参数作为进程结果，关闭文件，释放内存

+ 若父进程存活，保留结果直到父进程需要它，进入僵尸状态

  + 若僵尸状态时父进程死掉，该进程成为孤儿进程，会定期被清理

+ 否则释放所有数据结构和结果

  