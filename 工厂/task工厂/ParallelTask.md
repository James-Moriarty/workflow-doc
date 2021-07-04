## ParallelTask

并行任务，由[[SubTask基类]]派生而来，与`SubTask`也是友元类。

### 定义

~~~c++
class ParallelTask : public SubTask
{
public:
	ParallelTask(SubTask **subtasks, size_t n)
	{
		this->subtasks = subtasks;
		this->subtasks_nr = n;
	}

	SubTask **get_subtasks(size_t *n) const
	{
		*n = this->subtasks_nr;
		return this->subtasks;
	}

	void set_subtasks(SubTask **subtasks, size_t n)
	{
		this->subtasks = subtasks;
		this->subtasks_nr = n;
	}

public:
	// 特别注意这里实现了dispatch，如同SubTask中的dispatch，在这里启动了所有子任务
	// 同时，在dispatch中调用了subtask_done，这里也是友元类存在的意义
	virtual void dispatch() {
		SubTask **end = this->subtasks + this->subtasks_nr;
		SubTask **p = this->subtasks;

		this->nleft = this->subtasks_nr;
		if (this->nleft != 0)
		{
			do
			{
				// 设置其父节点
				(*p)->parent = this;
				// 设置entry？
				(*p)->entry = p;
				// 开始执行
				(*p)->dispatch();
			} while (++p != end);
		}
		else
			this->subtask_done();
	};

protected:
	// 任务数组
	SubTask **subtasks;
	// 任务个数
	size_t subtasks_nr;

private:	
	// 当前还有多少子任务再运行
	// 使用__sync_fetch_and_add来保证原子性
	size_t nleft;
	friend class SubTask;
};
~~~

### 并行任务结束

当所有子任务都执行完毕后，才会执行并行任务的回调函数。这个实现在[[SubTask基类]]中执行。

当执行完后，会判断其parent指针是不是为空，不为空则将nleft减一，当nleft为0时，则执行并行任务的回调函数。

