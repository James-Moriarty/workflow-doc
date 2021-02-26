## WFThreadTask

所属文件：factory/WFTask.h

单线线程计算任务，内部实现和[[WFGoTask]]基本类似，在这里不再赘述，仅简单介绍一些不同的地方。

### 定义

该任务和[[WFGoTask]]任务基本类似，区别在于`go`任务只是执行一个函数，而线程任务在创建的时候需要指定输入与输出：

```c++
template<class INPUT, class OUTPUT>
class WFThreadTask : public ExecRequest
{
...
}
```

执行函数定义如下：

```c++
std::function<void (INPUT *, OUTPUT *)> routine;
```

新建线程任务的时候，可设置回调函数，在回调函数中对计算结果进行处理。

## WFMultiThreadTask

所属文件：factory/WFTask.h

多线程任务，相当于多个单线程任务并行处理，内部也继承了`ParallelTask`来实现并行计算任务。

### 定义

```c++
template<class INPUT, class OUTPUT>
class WFMultiThreadTask : public ParallelTask
{
using Thread = WFThreadTask<INPUT, OUTPUT>;
public:	WFMultiThreadTask(Thread *const tasks[], size_t n,
			std::function<void (WFMultiThreadTask<INPUT, OUTPUT> *)>&& cb) :
		ParallelTask(new SubTask *[n], n),
		callback(std::move(cb)) {
		size_t i;

		for (i = 0; i < n; i++)
			this->subtasks[i] = tasks[i];

		this->user_data = NULL;
	}
}
```

从构造函数可见，在创建新的多线程任务时，需要提前创建好单线程任务。

