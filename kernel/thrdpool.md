## thrdpool

所属文件：kernel/thrdpool.h

线程池实现，兼容c的实现。

### 数据结构

```c++
struct __thrdpool {
	// 任务队列
	struct list_head task_queue;
	size_t nthreads;
	size_t stacksize;
	pthread_t tid;

	// 任务队列大锁
	pthread_mutex_t mutex;
	pthread_cond_t cond;

	
	pthread_key_t key;
	pthread_cond_t *terminate;
};
```

### 基本思路

创建线程后，等待任务到来。