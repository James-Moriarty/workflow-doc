## SubTask基类

所有的任务都从`SubTask`派生出来，该类为实现`workflow`中任务流最重要的类。

### 定义

首先看其定义：

```c++
class SubTask
{
public:
	// 主要用于触发任务
	virtual void dispatch() = 0;

private:
	// 用于执行回调函数
	virtual SubTask *done() = 0;

protected:
	void subtask_done();

public:
	ParallelTask *get_parent_task() const { return this->parent; }
	void *get_pointer() const { return this->pointer; }
	void set_pointer(void *pointer) { this->pointer = pointer; }

private:
	ParallelTask *parent;

	SubTask **entry;

	// pointer保存的是任务所属series的指针
	void *pointer;

public:
	SubTask()
	{
		this->parent = NULL;
		this->entry = NULL;
		this->pointer = NULL;
	}

	virtual ~SubTask() { }
	friend class ParallelTask;
};
```

### 重要对象

1. `pointer`指针，保存的是当前任务所属的任务流，在`workflow`中所有的任务都处于某个任务流中
2. `entry`指针
3. `parent`指针

### 重要函数

基类主要定义两个接口，一个是`dispatch()`，一个是`done()`。

1. `dispatch()`主要用于触发任务；
2. `done()`主要用于执行回调函数。

`subtask_done()`的定义如下，对于串行任务，会触发该任务的`done()`函数，并触发所属任务流中下一个任务的`dispatch()`。

```c++
void SubTask::subtask_done()
{
	SubTask *cur = this;
	ParallelTask *parent;
	SubTask **entry;

	while (1)
	{
		parent = cur->parent;
		entry = cur->entry;
		cur = cur->done();
		if (cur)
		{
			cur->parent = parent;
			cur->entry = entry;
			if (parent)
				*entry = cur;

			cur->dispatch();
		}
		else if (parent)
		{
			if (__sync_sub_and_fetch(&parent->nleft, 1) == 0)
			{
				cur = parent;
				continue;
			}
		}

		break;
	}
}
```