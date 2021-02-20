## Executor

所属文件：kernel/Executor.h

任务执行器，包含
1. ExecQueue 任务队列
2. ExecSession 任务会话
3. Executor 任务执行器

### ExceQueue

具体管理在[[WFGlobal#ExecManager]]进行管理，内部是一个双向链表实现队列

### ExecSession

任务执行会话基类

```c++
class ExecSession
{
private:
	// 运行时实际调用函数
	virtual void execute() = 0;
	virtual void handle(int state, int error) = 0;

protected:
	ExecQueue *get_queue() { return this->queue; }

private:
	ExecQueue *queue;

public:
	virtual ~ExecSession() { }
	friend class Executor;
};
```

### Executor

内部为[[thrdpool]]线程池。

```c++
class Executor {
public:
	int init(size_t nthreads);
	void deinit();
	
	// 新增任务
	int request(ExecSession *session, ExecQueue *queue);

private:
	struct __thrdpool *thrdpool;

private:
	// 执行函数
	static void executor_thread_routine(void *context);
	// 任务取消函数
	static void executor_cancel_tasks(const struct thrdpool_task *task);

public:
	virtual ~Executor() { }
};
```

当执行`request`增加任务时，增加的任务为：

```c++
struct thrdpool_task task = {
	.routine	=	Executor::executor_thread_routine,
	.context	=	queue
};
```

在`executor_thread_routine`中，取出任务队列中的头部任务，然后再将上述任务放入线程池中，直到任务队列中的任务全部被执行完。