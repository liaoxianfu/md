### go 调度模型



我们知道CPU执行速度四很快的，大部分的性能瓶颈不在cpu上而是在IO上，在大部分的系统中CPU都不是瓶颈，CPU的时间被大量的浪费了，增加CPU的有效时间是开发的重要目标。

为了增加CPU的有效时间有以下两种方法：

① 尽可能的让每个CPU的核心都有事情可做

在实际上要根据运行程序的特性（IO密集型还是CPU密集型）。合理的设置cpu和线程之间的关系。一般情况下线程数要大于CPU核心数，才能发挥机器的价值。

② 尽可能的提高每个CPU核心的做事效率

现代操作系统虽然能够进行并行调度，但是当进程数大于cpu的核心数时就存在进程切换的问题，切换就需要保存上下文，恢复堆栈。如果进程特别多就需要频繁的进行上下文的切换，这也很消耗时间。最理想的是让每个进程在切换时能够充分的使用CPU的分片时间。







