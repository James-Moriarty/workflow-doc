## WFHttpTask

### 定义及回调函数

~~~c++
using WFHttpTask = WFNetworkTask<protocol::HttpRequest,
								 protocol::HttpResponse>;
using http_callback_t = std::function<void (WFHttpTask *)>;
~~~

对于httptask而言，其就是一种网络任务，需要定义其request以及respnse。