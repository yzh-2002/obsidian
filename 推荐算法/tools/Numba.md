`numba`库：用于加速Python数值计算的JIT编译器，其可以将Python代码编译为快速的机器代码，设计目标是提高Numpy数组操作和科学计算的性能。

使用示例：在函数名上添加装饰器，例如：`@nb.jit(nopython = True, parallel = True, cache = True)`

## nopython
Numba的`jit`装饰器支持以两种编译模式运行：
1. `nopython`：会编译被装饰的函数，以便其完全运行而**不需要Python解释器的参与**，也是最佳实践，此时会获得最佳性能
2. `object`：如果`nopython`模式下编译失败，可使用`object`进行编译。如果`jit`未设置`nopython=True`，则Numba将识别它可以编译的循环，并将其编译成机器代码中运行的函数，对于其无法编译的循环，则交给Python解释器执行

Numba适合矩阵的并行计算，如果代码中出现其他库的函数（或者较为复杂的Python内置库，例如：`heapq`），Numba无法正常识别，最后只能将其交给Python解释器执行。对于此类函数，尽量不要使用`jit`装饰器。

## 并行方法

python多线程只适用于IO密集型场景，对于CPU密集型场景，python多线程效率和单线程效率一样，原因在于：`GIL(Global interpreter Lock)`，简单来讲：每个线程想要执行必须先获得这把锁，也就导致每个时刻只能有一个线程执行。

故先前`Numba`想要并发执行需要设置`nopython=True`，也即绕过Python解释器。

基于此，对Python而言，CPU密集型任务只能依赖于多进程。

Python多进程在创建新进程时，有三种模式：
1. `spawn`：`windows`可用，其会复制主进程所需环境
2. `fork`：仅`linux`下可用，子进程为父进程的副本，共享父进程的所有内存空间（不需要额外的内存分配和数据复制），仅在子进程修改内存时才会复制数据（`Copy on Write`）
3. `forkserver`：仅`linux`下可用

