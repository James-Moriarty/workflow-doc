## ParallelWork

所属文件：factory/Workflow.h

并行任务，其可由多个串行组成，当所有串行任务都执行完毕，执行所设置的callback；

### 定义

~~~c++
class ParallelWork : public ParallelTask {
public:
	void start() {
		assert(!series_of(this));
		// 在这里，将并行任务放入一个series，从这里也可以看出，并行任务也是一个series
		Workflow::start_series_work(this, nullptr);
	}

	void dismiss() {
		assert(!series_of(this));
		this->dismiss_recursive();
	}

public:
	void add_series(SeriesWork *series);

public:
	void *get_context() const { return this->context; }
	void set_context(void *context) { this->context = context; }

public:
	const SeriesWork *series_at(size_t index) const {
		if (index < this->subtasks_nr)
			return this->all_series[index];
		else
			return NULL;
	}

	const SeriesWork& operator[] (size_t index) const {
		return *this->series_at(index);
	}

	size_t size() const { return this->subtasks_nr; }

public:
	void set_callback(parallel_callback_t callback) {
		this->callback = std::move(callback);
	}

protected:
	virtual SubTask *done();

protected:
	void *context;
	parallel_callback_t callback;

private:
	void expand_buf();
	// 递归取消后续所有任务
	void dismiss_recursive();

private:
	size_t buf_size;
	SeriesWork **all_series;

protected:
	ParallelWork(parallel_callback_t&& callback);
	ParallelWork(SeriesWork *const all_series[], size_t n,
				 parallel_callback_t&& callback);
	virtual ~ParallelWork() { delete []this->subtasks; }
	friend class SeriesWork;
	friend class Workflow;
};
~~~

### 使用

使用Workflow工厂创建并行任务。

~~~c++
class Workflow {
public:

	static ParallelWork *
	create_parallel_work(parallel_callback_t callback);

	static ParallelWork *
	create_parallel_work(SeriesWork *const all_series[], size_t n,
						 parallel_callback_t callback);

	static void
	start_parallel_work(SeriesWork *const all_series[], size_t n,
						parallel_callback_t callback)
};
~~~

从创建中可以看出，需要向其传入SeriesWork数组，其实也是子任务。