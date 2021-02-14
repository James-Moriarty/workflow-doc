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
	void (*cb)(struct poller_result *, void *);
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
	// 定时器链表
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

### timerfd

对于定时器任务，采用了绝对时间的方式，每次`epoll_wait`之前都会将红黑树或链表将最近的`timer`添加到`timerfd`中。