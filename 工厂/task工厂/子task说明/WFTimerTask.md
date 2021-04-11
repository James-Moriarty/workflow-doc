## WFTimerTask

所属文件：factory/WFTask.h

### 继承关系

![[Pasted image 20210215150152.png]]

### 回调函数

```c++
using timer_callback_t = std::function<void (WFTimerTask *)>;
```

### 执行关系

创建`timer`任务后，会调用[[Communicator]]的`sleep`方法，新建一个`timer`任务，并将其加入到[[mpoller]]中，等待任务超时。

当[[mpoller]]等到任务超时后，会执行`callback`，将`timer`任务放入[[Communicator]]的线程池中等待执行。

如此一来，就实现了`timer`的异步，任务到时间后都放到线程池中等待计算。

### 开始运行

创建任务后，需要执行`start()`来开始运行任务，一般在这个时候，将任务放入`series`中，然后开始运行。