## Communicator

所属文件：kernel/Communicator.h

### 初始化 init

`CommScheduler::init(size_t poller_threads, size_t handler_threads)`

初始化的时候需要`poller`的线程数以及`handler`的线程数

在初始化阶段主要初始化三个比较重要的成员：

```c++
class Communicator {	
private:
	struct __mpoller *mpoller;
	struct __msgqueue *queue;
	struct __thrdpool *thrdpool;
}
```

执行顺序为：

1. msgqueue_create

`queue`为内部定义的消息队列，采用消费生产双队列的模式，具体说明见[[msgqueue]]。

默认队列长度为4096，偏移量为`sizeof(struct poller_result)`

2. mpoller_create

创建`poller`

3. mpoller_start

启动`poller`

4. create_handler_threads

创建`handler`，实际为创建线程池

5. thread_schedule

启动线程池。