## SeriesWork

所属文件：factory/Workflow.h

`SeriesWork`为串行任务流，可以把其看做是一个流水线任务。其内部采用了一个环形数组来实现任务的控制。

### 定义

首先看其定义

```c++
class SeriesWork
{
public:
	void start() {
		assert(!this->in_parallel);
		this->first->dispatch();
	}

	/* Call dismiss() only when you don't want to start a created series.
	 * This operation is recursive, so only call on the "root". */
	void dismiss() {
		assert(!this->in_parallel);
		this->dismiss_recursive();
	}

public:
	// 插入新任务
	void push_back(SubTask *task);
	void push_front(SubTask *task);

public:
	void *get_context() const { return this->context; }
	void set_context(void *context) { this->context = context; }

public:
	/* Cancel a running series. Typically, called in the callback of a task
	 * that belongs to the series. All subsequent tasks in the series will be
	 * destroyed immediately and recursively (ParallelWork), without callback.
	 * But the callback of this canceled series will still be called. */
	void cancel() { this->canceled = true; }

	/* Parallel work's callback may check the cancellation state of each
	 * sub-series, and cancel it's super-series recursively. */
	bool is_canceled() const { return this->canceled; }

public:
	void set_callback(series_callback_t callback) {
		this->callback = std::move(callback);
	}

public:
	/* The next 3 methods are intended for task implementations only. */
	SubTask *pop();

	// 设置最后一个任务
	void set_last_task(SubTask *last) {
		last->set_pointer(this);
		this->last = last;
	}

	// 取消最后一个任务
	void unset_last_task() { this->last = NULL; }

protected:
	void *context;
	series_callback_t callback;

private:
	// 返回下一个任务
	SubTask *pop_task();
	// 扩大环形数组，2倍扩大
	void expand_queue();
	void dismiss_recursive();

private:
	// 第一个任务
	SubTask *first;
	// 最后一个任务
	SubTask *last;

	// 环形数组
	SubTask **queue;
	// 环形数组大小
	int queue_size;
	// 环形数组头部
	int front;
	// 环形数组尾部
	int back;

	bool in_parallel;
	bool canceled;

	// 环形数组访问锁
	std::mutex mutex;

protected:
	SeriesWork(SubTask *first, series_callback_t&& callback);
	virtual ~SeriesWork() { delete []this->queue; }
	friend class ParallelWork;
	friend class Workflow;
};
```

其中，特别注意：

1. `first`和`last`任务并不在环形队列中，也就是说，所设置的`last`任务肯定是最后一个执行的任务；

### 获取某一个任务所处的任务流

从定义中也可以看出，往任务流中添加任务时，会调用[[SubTask基类]]中的`set_pointer()`，在每个任务中保存其所属的任务流，因此获取当前任务流可以通过这种方法来实现。

`workflow`中也提供了全局方法：

```c++
static inline SeriesWork *series_of(const SubTask *task)
{
	return (SeriesWork *)task->get_pointer();
}

static inline SeriesWork& operator *(const SubTask& task)
{
	return *series_of(&task);
}
```

向任务流中新增任务时，除了调用`push_*()`接口外，也可以调用`c++`中的`<<`运算符。

```c++
static inline SeriesWork& operator << (SeriesWork& series, SubTask *task)
{
	series.push_back(task);
	return series;
}
```