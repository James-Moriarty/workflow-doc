## WFHttpTask

### 定义及回调函数

~~~c++
using WFHttpTask = WFNetworkTask<protocol::HttpRequest,
								 protocol::HttpResponse>;
using http_callback_t = std::function<void (WFHttpTask *)>;
~~~

对于httptask而言，其就是一种网络任务，需要定义其request以及respnse。

### 继承关系

![[Pasted image 20210411203901.png]]

特别注意：

1. 实际工厂返回给用户的是WFNetWorkTask这一层，这层包括一些get接口提供给用户进行操作
2. 最顶层的类为session，可以看出client任务本质一个session，在Communicator中会保存session，相当于保存上下文，主要操作在Request中，在这层会将请求放入Communicator中，并发送给请求方
3. ComplexClientTask中增加了一部分接口，如route()，finish_once()等接口，也增加了用户扩展的可能性。


### 执行

执行顺序为：

1. 创建新请求，设置req和resp
2. WFComplexClientTask执行dispatch()
3. 调用CommRequest的dispatch()
4. 检查是否超过max_connection，超过max_connection直接返回错误（35），当请求量比较大时，可以考虑设置wait_timeout来等待一会保证成功率；
5. 放入communicator
6. 调用message_out()获取发送req
7. 请求远端
8. 远端返回，调用message_in()写入resp，调用用户回调函数
9. 结束

对于ComplexClientTask，基本流程如上所述，如果比较复杂，比如需要重定向，会在client任务之前再增加一个route任务，用于重定向。