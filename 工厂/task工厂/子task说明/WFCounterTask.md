## WFCounterTask

所属文件：factory/WFTask.h
命名计数器：factory/WFTaskFactory.cc

`counter task`有匿名计数器和命名计数器两种，具体使用和区别见[[计数器任务]]。

### 继承关系

![[Pasted image 20210208132455.png]]

基类：[[SubTask基类]]

### 回调函数

```c++
using counter_callback_t = std::function<void (WFCounterTask *)>;
```

### 匿名计数器

匿名计数器任务创建:

```c++
class WFTaskFactory {
	...
public:
	/* Create an unnamed counter. Call counter->count() directly.
	 * NOTE: never call count() exceeding target_value. */
	static WFCounterTask *create_counter_task(unsigned int target_value,
											  counter_callback_t callback) {
		return new WFCounterTask(target_value, std::move(callback));
	}
	...
}
```

### WFCounterTask

使用上述接口后，会得到一个`task`实例，其中该类的内容也比较简单：

```c++
class WFCounterTask : public WFGenericTask
{
public:
	virtual void count()
	{
		if (--this->value == 0)
		{
			this->state = WFT_STATE_SUCCESS;
			this->subtask_done();
		}
	}

public:
	void set_callback(std::function<void (WFCounterTask *)> cb)
	{
		this->callback = std::move(cb);
	}

protected:
	virtual void dispatch()
	{
		this->WFCounterTask::count();
	}

	virtual SubTask *done()
	{
		SeriesWork *series = series_of(this);

		if (this->callback)
			this->callback(this);

		delete this;
		return series->pop();
	}

protected:
	std::atomic<unsigned int> value;
	std::function<void (WFCounterTask *)> callback;

public:
	WFCounterTask(unsigned int target_value,
				  std::function<void (WFCounterTask *)>&& cb) :
		value(target_value + 1),
		callback(std::move(cb))
	{
	}

protected:
	virtual ~WFCounterTask() { }
};
```

内部计数使用一个原子量，每执行一次`count()`就减1，当计数为0时，则执行回调函数。

> 可以看到在初始化的时候，内部存储的`value`比实际大小大一，其在`dispatch()`执行了减一操作，也就是开始执行任务。

### 命名计数器

除了上述的匿名计数器，还提供了命名计数器，其存在的意义就是为了解决`count`次数超过指定次数后的野指针的问题。从上述的类方法`done`中，可以看到，执行了`delete`把当前实例进行了删除，所以当`count`到达指定次数后，在执行`count()`就会出现问题。

其新建`task`接口如下所述。

```c++
class WFTaskFactory {
public:
	/* Create a named counter. */
	// counter_name 计数器名称
	static WFCounterTask *create_counter_task(const std::string& counter_name,
											  unsigned int target_value,
											  counter_callback_t callback);

	/* Count by a counter's name. When count_by_name(), it's safe to count
	 * exceeding target_value. When multiple counters share a same name,
	 * this operation will be performed on the first created. If no counter
	 * matches the name, nothing is performed. */
	static void count_by_name(const std::string& counter_name) {
		WFTaskFactory::count_by_name(counter_name, 1);
	}

	/* Count by name with a value n. When multiple counters share this name,
	 * the operation is performed on the counters in the sequence of its
	 * creation, and more than one counter may reach target value. */
	static void count_by_name(const std::string& counter_name, unsigned int n);
}
```

内部使用红黑树加链表来实现命名计数器中的存储。特别注意该红黑树为全局可见，内部使用一把大锁来保证读写一致性。

> 特别注意：
> 可以发现在`WFCounterTask`的定义中，`count`是一个虚函数，设置为虚函数的原因是因为，在命名函数中，也可以通过`count()`来计数，其内部对`count`进行了重写；当然，使用`count_by_name`来进行计数也是一样的。