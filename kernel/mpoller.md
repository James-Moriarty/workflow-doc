## mpoller

`io`以及`timer`部分的控制都放在了`mpoller`中，`mpoller`的结构如下所示：

```c++
struct __mpoller {
	unsigned int nthreads;
	poller_t *poller[1];
};
```

内部包含多个`poller`，在创建的时候设置线程数。

`mpoller`主要有以下操作：

```c++
// 创建mpoller
mpoller_t *mpoller_create(const struct poller_params *params, size_t nthreads);
// 启动mpoller
int mpoller_start(mpoller_t *mpoller);
// 停止mpoller
void mpoller_stop(mpoller_t *mpoller);
// 摧毁mpoller
void mpoller_destroy(mpoller_t *mpoller);

// 新增
static inline int mpoller_add(const struct poller_data *data, int timeout, mpoller_t *mpoller);
// 删除
static inline int mpoller_del(int fd, mpoller_t *mpoller);
// 修改
static inline int mpoller_mod(const struct poller_data *data, int timeout, mpoller_t *mpoller);
// 新增超时fd
static inline int mpoller_set_timeout(int fd, int timeout, mpoller_t *mpoller);
// 新增timer
static inline int mpoller_add_timer(const struct timespec *value, void *context, mpoller_t *mpoller);
```

该部分主要为`poller`的封装，当为多线程的时候，会根据`fd`值取模分配到指定的`poller`上。以下主要对`poller`进行说明

## poller

### 数据结构

```c++
struct __poller {
	// 文件描述符数量
	size_t max_open_files;
	// 各类回调函数
	poller_message_t *(*create_message)(void *);
	int (*partial_written)(size_t, void *);
	
	// 回调函数，执行communicator的callback，本质是将任务放到所属communicator的线程池中。
	void (*cb)(struct poller_result *, void *);
	// 所属的communicator
	void *ctx;

	// 各类fd
	pthread_t tid;
	// epoll fd
	int pfd;
	// timer fd
	int timerfd;

	int pipe_rd;
	int pipe_wr;
	int stopped;

	// 定时器红黑树
	struct rb_root timeo_tree;
	// 定时器红黑树头结点
	struct rb_node *tree_first;

	// 定时器链表 存放已经超时的timer，每次epoll wait后遍历所有timer
	struct list_head timeo_list;

	struct list_head no_timeo_list;
	struct __poller_node **nodes;
	pthread_mutex_t mutex;
	char buf[POLLER_BUFSIZE];
};
```

### 创建

1. 按照最大描述符数量创建`nodes`数组；
2. 使用`epoll create`创建`epoll fd`
3. 使用`timerfd_create`创建`timerfd`
4. 把`timerfd`加入`epoll`监听`fd`
5. 创建新线程开始监听

### 定时任务

1. 调用`poller_add_timer`增加timer计时器；
2. 新建`poller_node`，并将节点插入`poller`
3. 其中，节点的timeout中存储着绝对时间，而不是相对的超时时间
4. 首先对比`poller`中定时器队列，如果比最后一个定时器的时间还大，则插入队列尾部
5. 如果不是，则将节点插入到红黑树中
6. 如果节点被插入到队列中，则和红黑树的第一个节点对比；如果节点被插入到红黑树中，则和队列中第一个元素进行对比
7. 如果不存在这些节点，那么需要插入新节点，如果新节点的时间比以上节点的时间小，那么要插入新节点
8. 在linux中，使用timerfd来控制时间参数
9. epoll没获取到任务，都会根据当前时间，将已经超时的时间任务拿出来，并放到handler执行回调函数
