## WFGoTask

所属文件：factory/WFTask.h

### 继承关系

![[Pasted image 20210219121948.png]]

### queue and executor

在go任务中，比较重要的两个内容是任务队列以及执行器。

在创建go任务的时候，需要指明任务队列的名称。

```c++
template<class FUNC, class... ARGS>
inline WFGoTask *WFTaskFactory::create_go_task(const std::string& queue_name,
											   FUNC&& func, ARGS&&... args)
{
	auto&& tmp = std::bind(std::forward<FUNC>(func),
						   std::forward<ARGS>(args)...);
	// 获取队列以及执行器
	return new __WFGoTask(WFGlobal::get_exec_queue(queue_name),
						  WFGlobal::get_compute_executor(),
						  std::move(tmp));
}
```

内部获取到[[Executor]]后，将该任务放入队列中，等待线程池执行任务。