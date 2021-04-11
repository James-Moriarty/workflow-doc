## WFNetworkTaskFactory

所属文件：factory/WFTaskFactory.h

### HttpTask任务

~~~c++
static WFHttpTask *create_http_task(const std::string& url,
									int redirect_max,
									int retry_max,
									http_callback_t callback);

static WFHttpTask *create_http_task(const ParsedURI& uri,
									int redirect_max,
									int retry_max,
									http_callback_t callback);
~~~