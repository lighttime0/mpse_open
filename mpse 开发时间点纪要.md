# mpse 开发时间点纪要



###2019.02.25 - 2019.03.01

根据上学期初次会议的讨论，完成Cloud9与Klee的对比实验，确定初期将Cloud9移植到Klee-1.4上的方案



### 2019.03.04 - 2019.03.15

移植Cloud9，这两周的移植工作下来，感觉之前预期的需要一个月的时间还是比较准的。



## 2019.03.18 - 2019.03.29

Cloud9第一轮移植代码大部分完成了，开始修改CMakeLists.txt。



目前对Cloud9有一个比较整体的把握了。Cloud9对Klee的代码修改主要目标有两个：

（1）支持分布式符号执行；

（2）支持多线程、多进程。



Cloud9的修改主要有三部分：

（1）代码的Redistribution。目的是开发分布式符号执行，分布式符号执行中，需要每台机器维护一个独立的符号执行流程，同时还要彼此通信，所以需要将一些代码重新分布，让分布式的代码可以方便地掉用。

（2）代码的增加。目的是增加分布式、多进程、多线程的功能。分布式的代码我们不关心，多进程、多线程的代码大致的思路是：

* 增加对fork、pthread_xxx等函数的建模；增加pipe、event等机制的建模；
* 在符号执行的过程中，对每个线程的相关信息记录到ExecutionState中，继承之前的约束后，对每个线程单独记录约束条件，并求解。


移植的整体思路是：

（1）以Klee-1.4为base，接口有冲突的时候尽量以Klee-1.4的代码为基准。Cloud9对代码的Redistribution也尽量去掉，以Klee-1.4的distirbution为基准。

（2）分布式符号执行部分的代码尽量剔除掉，减少潜在的bug。

（3）runtime/POSIX中和多线程支持无关的，且比较复杂的代码，暂时不动。这部分代码Cloud9改的面目全非了，移植的话耗时较多，但是增加的功能也就只有socket和pipe的支持，所以目前先不动这块的代码，直接用Klee-1.4的代码。



## 2019.04.01 - 2019.04.24

修改代码，目前可以编译通过。

在另一台机器上测试编译，证明可以通过[文档](https://github.com/lighttime0/mpse/blob/master/code/Klee编译及Cloud9的移植环境准备.md)在docker中复现。



目前的编译使用了一些方法来使编译通过，这些方法在以后都需要改掉，包括：

（1）Cloud9使用Crypto++库来定义fork_id，理论上来说不需要这个库的，但是并不重要，所以就先用了。具体的记录在[这里](https://github.com/lighttime0/mpse_open/blob/master/编译中遇到的问题.md)。

（2）runtime的POSIX目录下文件目前用的还是Klee-1.4的，Cloud9的POSIX目录下的文件问题太多了，目前还处在试验阶段。POSIX目录下的文件定义了POSIX的接口实现，pthread_create()等函数的建模就是在这个文件夹里，所以目前还无法测试pthread_create()函数的功能。



## 2019.04.26 - 2019.05.09

这段时间主要是修改运行时遇到的BUG。

### Bug 1

昨天解决的一个Bug记录在[这里](https://github.com/lighttime0/mpse_open/blob/master/dev-doc/Debug记录/2019.05.01.md)。

这个Bug解决后，可以用运行没有符号量的最简单的testcase：

```bash
root@server1:/home/chy/secProjects/lt_workspace/mpse/mpse_build# bin/klee -libc=uclibc -posix-runtime ../testcases/klee_testcases_from_huawei/bitcode_llvm34/pthread_base1_sp.bc "hahaha" 6
KLEE: NOTE: Using POSIX model: /home/chy/secProjects/lt_workspace/mpse/mpse_build/Debug+Asserts/lib/libkleeRuntimePOSIX.bca
KLEE: NOTE: Using klee-uclibc : /home/chy/secProjects/lt_workspace/mpse/mpse_build/Debug+Asserts/lib/klee-uclibc.bca
KLEE: output directory is "/home/chy/secProjects/lt_workspace/mpse/mpse_build/../testcases/klee_testcases_from_huawei/bitcode_llvm34/klee-out-66"
KLEE: Using STP solver backend
KLEE: WARNING ONCE: calling external: syscall(16, 0, 21505, 77744016) at /home/chy/secProjects/lt_workspace/mpse/mpse-git/code/c9-porting/runtime/POSIX/fd.c:980
KLEE: WARNING ONCE: calling __user_main with extra arguments.
KLEE: WARNING ONCE: Alignment of memory from call "malloc" is not modelled. Using alignment of 8.
KLEE: ERROR: /home/chy/secProjects/lt_workspace/klee/klee-uclibc/libc/string/memcpy.c:29: memory error: out of bound pointer
KLEE: NOTE: now ignoring this error at this location

KLEE: done: total instructions = 15093
KLEE: done: completed paths = 1
KLEE: done: generated tests = 1
```



### Bug 2

这个Bug昨天刚把现象整理出来，还没解决，记录在[这里](https://github.com/lighttime0/mpse_open/blob/master/dev-doc/Debug记录/2019.05.08.md)。



