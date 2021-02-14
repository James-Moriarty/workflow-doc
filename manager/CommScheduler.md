## CommScheduler

所属文件：kernel/CommScheduler.h

该类也是一个接口类，主要操作都在`class Communicator`中，具体可见[[Communicator]]说明。

### 初始化

初始化的时候指定`poller`和`handler`线程数，然后调用`Communicator`的初始化方法。

```c++
int init(size_t poller_threads, size_t handler_threads)
```

