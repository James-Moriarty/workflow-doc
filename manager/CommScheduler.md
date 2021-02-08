## CommScheduler

所属文件：kernel/CommScheduler.h

该类也是一个接口类，主要操作都在`class Communicator`中，所以一下介绍也是在介绍`Communication`类

### 初始化 init

`CommScheduler::init(size_t poller_threads, size_t handler_threads)`

初始化的时候需要`poller`的线程数以及`handler`的线程数

比较重要的三个成员为：

```c++
class Communicator {	
private:
	struct __mpoller *mpoller;
	struct __msgqueue *queue;
	struct __thrdpool *thrdpool;
}
```