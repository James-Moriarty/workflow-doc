## thrdpool

所属文件：kernel/thrdpool.h

线程池实现，兼容c的实现。

### 线程池结构

```c++
struct __thrdpool {
	// 任务队列
	struct list_head task_queue;
	
	// 线程数
	size_t nthreads;
	size_t stacksize;
	pthread_t tid;

	// 任务队列锁
	pthread_mutex_t mutex;
	pthread_cond_t cond;

	// 判断当前线程是否在线程池中
	pthread_key_t key;
	// 终止标识
	pthread_cond_t *terminate;
};
```

### 线程池任务结构

```c++
struct thrdpool_task {
	// 运行函数指针
	void (*routine)(void *);
	// 上下文
	void *context;
};
```

当线程池拿到任务后，`routine(context)`这样来执行任务。

### 接口

```c++
// 创建线程池
thrdpool_t *thrdpool_create(size_t nthreads, size_t stacksize);
// 新增任务
int thrdpool_schedule(const struct thrdpool_task *task, thrdpool_t *pool);
// 增加线程数量
int thrdpool_increase(thrdpool_t *pool);
// 判断线程池是否在运行
int thrdpool_in_pool(thrdpool_t *pool);
// 摧毁线程池
void thrdpool_destroy(void (*pending)(const struct thrdpool_task *), thrdpool_t *pool);
```


### pthread_key_t

结构体中另外保存了一个`key`，这个`key`的作用就是用来判断当前任务所在的线程是否处于线程池中，在[[WFFuture]]中会用到该`key`来判断是否动态增加线程。

