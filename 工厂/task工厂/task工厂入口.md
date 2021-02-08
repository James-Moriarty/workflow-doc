## 工厂入口

所有的`task`创建需要通过工厂来建立，主要分为以下三个工厂

### [[WFTaskFactory]]

该工厂类主要为一些基本任务的创建工厂，如：

1. http task
2. redis task
3. io task
4. timer task
5. counter task
6. graph task
7. dns task

### [[WFNetworkTaskFactory]]

该工厂主要为网络任务的创建工厂，在调用时需要显式声明`req`以及`respond`类型。

### [[WFThreadTaskFactory]]

该工厂主要为线程任务的创建工厂，基本可以认为是线程池，但又不局限于简单的线程池。同样，在使用该工厂时需要显示声明输入与输出类型。


