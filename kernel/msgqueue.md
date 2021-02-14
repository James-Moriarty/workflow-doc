## msgqueue 消息队列

所属文件：kernel/msgqueue.h

此类为兼容c的消息队列实现，采用了双队列的方式来实现，一个队列用来`put`消息，另外一个队列用来`get`消息。

### 初始化

```c++
/* 创建消息队列
 * @param maxlen 为最大消息数量
 * @param linkoff 为消息 header 的偏移量
 */
msgqueue_t *msgqueue_create(size_t maxlen, int linkoff);
```

在初始化的时候，需要指定最大消息数量与消息头部偏移量；

最大消息数量比较好理解，需要注意的是，该消息队列采用了*双队列*的实现，实际消息队列中的最大消息数量为用户指定最大数量的两倍。

#### 队列内部链表实现

消息头部偏移量存在的目的主要是为了利用`msg`中的空间，来形成链表。因为对于一个消息队列的基本结构来说，应该是下述结构：

```c++
// 常规的消息队列节点
struct Node {
	void* your_message;
	Node* next;
};
```

但是在这里没有指定节点内容，而是使用了`msg`头部偏移后的空间，因此在使用的时候，该偏移量的后部需要保留一个指针大小的空间，以为消息队列中使用，具体可见下图。

![[Pasted image 20210214153220.png]]

### `put`数据

放数据的时候分为堵塞和非堵塞两种方式，各自有不同的应用场景。

#### 堵塞

默认为堵塞，在`put`数据的时候，如果目前消息队列已超过设定的最大值，那么就堵塞等待消息被消费。

#### 非堵塞

如果设置了非堵塞，会释放`get_cond`以及`put_cond`，通知目前堵塞的线程继续往下执行。且此时用户设置的最大值不再生效。即会继续向`put`队列中放入数据。

#### 存放数据

在[[#队列内部链表实现]]中指定了一个`linkoff`值，实际使用方法为：

```c++
// link 为实际的地址
// *link 为二级指针，该内容为msg中的空间，因此在头文件的说明中，需要在计算偏离后的大小至少为一个指针的大小

void **link = (void **)((char *)msg + queue->linkoff);
*link = NULL;
```

### `get`数据

1. 如果当前`get`队列中还有数据，那么直接读取队列中下一个数据；
2. 如果当前`get`队列中没有数据，那么尝试交换`get`队列和`put`队列。

`get`数据的时候，返回消息的头部：

```c++
// 获取消息头部
msg = (char *)*queue->get_head - queue->linkoff;
// 将头部移动到下一个元素上
*queue->get_head = *(void **)*queue->get_head;
```

### 数据结构

```c++
struct __msgqueue {
	size_t msg_max;
	size_t msg_cnt;

	// 消息的偏移量
	int linkoff;
	// 是否为堵塞
	int nonblock;

	// 消息队列的两个头部
	void *head1;
	void *head2;

	// get 队列
	void **get_head;

	// put 队列头部和尾部
	void **put_head;
	void **put_tail;

	// 锁
	pthread_mutex_t get_mutex;
	pthread_mutex_t put_mutex;
	pthread_cond_t get_cond;
	pthread_cond_t put_cond;
};
```

### 基本接口

```c++
// 创建
msgqueue_t *msgqueue_create(size_t maxlen, int linkoff);
// put数据
void msgqueue_put(void *msg, msgqueue_t *queue);
// get数据
void *msgqueue_get(msgqueue_t *queue);

// 设置为不堵塞
void msgqueue_set_nonblock(msgqueue_t *queue);
// 设置为堵塞
void msgqueue_set_block(msgqueue_t *queue);
// 摧毁队列
void msgqueue_destroy(msgqueue_t *queue);
```
