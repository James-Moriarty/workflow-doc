## WFTaskFactory

所属文件：factory/WFTaskFactory.h

在该类中，可创建多种不同的`task`，如：

1. http task
2. redis task
3. io task
4. timer task
5. counter task
6. graph task
7. dns task

基本涵盖了大部分的使用场景。以下对各类`task`进行简单的介绍。

PS：所有的任务都继承于[[SubTask基类]]

### 计数器任务

计数器任务，具体的应用场景可见[[计数器任务]]，具体的说明见[[WFCounterTask]]。

```c++
class WFTaskFactory {
public:
	/* Counter is like semaphore. The callback of counter is called when
	 * 'count' operations reach target_value & after the task is started.
	 * It's perfectly legal to call 'count' before the task is started. */

	/* Create an unnamed counter. Call counter->count() directly.
	 * NOTE: never call count() exceeding target_value. */
	static WFCounterTask *create_counter_task(unsigned int target_value,
											  counter_callback_t callback)
	{
		return new WFCounterTask(target_value, std::move(callback));
	}

	/* Create a named counter. */
	static WFCounterTask *create_counter_task(const std::string& counter_name,
											  unsigned int target_value,
											  counter_callback_t callback);

	/* Count by a counter's name. When count_by_name(), it's safe to count
	 * exceeding target_value. When multiple counters share a same name,
	 * this operation will be performed on the first created. If no counter
	 * matches the name, nothing is performed. */
	static void count_by_name(const std::string& counter_name)
	{
		WFTaskFactory::count_by_name(counter_name, 1);
	}

	/* Count by name with a value n. When multiple counters share this name,
	 * the operation is performed on the counters in the sequence of its
	 * creation, and more than one counter may reach target value. */
	static void count_by_name(const std::string& counter_name, unsigned int n);
}
```

### 定时任务

定时任务需要传入定时时间以及回调函数，具体内容及实现可见[[WFTimerTask]]

```c++
class WFTaskFactory {
public:
	static WFTimerTask *create_timer_task(unsigned int microseconds,
										  timer_callback_t callback);

	/* 当前没用 */
	static WFTimerTask *create_timer_task(const std::string& timer_name,
										  unsigned int microseconds,
										  timer_callback_t callback);
}
```

### go任务

`go`任务类似`golang语言中的`go`命令，即：

```go
func go_task() {
	sleep(5)
}

func main() {
	...
	go go_task
	...
}
```

`golang`是这样来实现异步操作的，在`workflow`中也提供了这种任务方式，具体细节可见[[WFGoTask]]

```c++
class WFTaskFactory {
public:
	template<class FUNC, class... ARGS>
	static WFGoTask *create_go_task(const std::string& queue_name,
									FUNC&& func, ARGS&&... args);
}
```

另外需要特别注意的是，go任务没有回调函数。