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

创建[[mpoller]]，主要作用为`timer`以及`io`相关的监听。

3. mpoller_start

启动[[mpoller]]。

4. create_handler_threads

创建`handler`，实际为创建[[thrdpool]]，用来处理各类任务，即线程池。

5. thread_schedule

启动线程池。

### timer任务相关内容

对于一个[[WFTimerTask]]任务，会新增一个`timer`到[[mpoller]]中。同时保留该`timer`的上下文。

### 运行

两个线程池 mpoller 和 handler；

mpoller用于接收发送任务，同时也有定时任务；

mpoller接收到任务后会将任务放入消息队列，handler用来处理后续的任务，一般来讲耗时的任务放入handler来处理；如果有更加重的任务，需要放到运算线程中进行处理。

### mpoller

创建poller时，会对poller的参数进行配置

```c++
struct poller_params params = {
	.max_open_files		=	65535,
	.create_message		=	Communicator::create_message,
	.partial_written	=	Communicator::partial_written,
	.callback			=	Communicator::callback,
	.context			=	this
};
```

#### `create_message`

对于每一个链接，都有一个session来保存上下文，在poller接收到请求或返回内容时，会调用create_message函数来写数据，一般是调用session的`message_in()`获取存放数据的buffer（PS：发送时则调用`message_out()`）。

#### `partial_written`

#### `callback`

当执行完毕时，poller会调用callback，其实是将上下文放入Communicator的队列，等待handler进行处理请求。