## WFThreadTaskFactory

所属文件：factory/WFTaskFactory.h

### 主要接口

该工厂主要用于创建线计算任务，接口如下：

```c++
template<class INPUT, class OUTPUT>
class WFThreadTaskFactory {
private:
	using T = WFThreadTask<INPUT, OUTPUT>;
	using MT = WFMultiThreadTask<INPUT, OUTPUT>;

public:
	static T *create_thread_task(const std::string& queue_name,
								 std::function<void (INPUT *, OUTPUT *)> routine,
								 std::function<void (T *)> callback);

	static MT *create_multi_thread_task(const std::string& queue_name,
										std::function<void (INPUT *, OUTPUT *)> routine,
										size_t nthreads,
										std::function<void (MT *)> callback);

public:
	static T *create_thread_task(ExecQueue *queue, Executor *executor,
								 std::function<void (INPUT *, OUTPUT *)> routine,
								 std::function<void (T *)> callback);
};
```

和[[WFTaskFactory#go任务]]任务存在区别，`go`任务中只要输入函数与参数即可。

在`threadtask`中需要对`input`和`output`进行声明，同时还可以增加`callback`。

### 创建单线程任务

使用`create_thread_task()`来创建单线程任务。

### 创建多线程任务

使用`create_multi_thread_task()`来创建多线程任务。