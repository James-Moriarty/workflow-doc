## go task

`workflow`中提供了一种更简单的使用计算任务的方法，模仿`go`语言实现的`go task`，使用`go task`来实现任务无需定义输入与输出，所有数据通过函数参数传递。

### 创建`go task`

```c++
class WFTaskFactory
{
    ...
public:
    template<class FUNC, class... ARGS>
    static WFGoTask *create_go_task(const std::string& queue_name,
                                    FUNC&& func, ARGS&&... args);
};
```

### 示例

异步实现一个加法函数，并且在函数运行结束后打印出结果，可以这样实现：

```c++
#include <stdio.h>
#include <utility>
#include "workflow/WFTaskFactory.h"
#include "workflow/WFFacilities.h"

void add(int a, int b, int& res)
{
    res = a + b;
}

int main(void)
{
    WFFacilities::WaitGroup wait_group(1);
    int a = 1;
    int b = 1;
    int res;

    WFGoTask *task = WFTaskFactory::create_go_task("test", add, a, b, std::ref(res));
    task->set_callback([&](WFGoTask *task) {
        printf("%d + %d = %d\n", a, b, res);
        wait_group.done();
    });
 
    task->start();
    wait_group.wait();
    return 0;
}
```

其中`go task`的使用与其他任务没有太大区别，也有`user_data`域可以使用，唯一一点不同的是，`go task`创建时不传`callback`，但是可以和其他任务一样可以`set_callback`，如果go task函数的某个参数是引用，需要使用std::ref，否则会变成值传递，这是c++11的特征。

### 线程池用法

用户可以只使用`go task`，这样可以将`workflow`退化成一个线程池，而且线程数量默认等于机器cpu数。  
但是这个线程池比一般的线程池又有更多的功能，比如每个任务有`queue name`，任务之间还可以组成各种串并联或更复杂的依赖关系。