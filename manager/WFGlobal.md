## WFGlobal

所属文件：manager/WFGlobal.h

WFGlobal中保存这各种配置，并提供了各种调度器的接口，所有需要的调度器都从该类获取。

```c++
/**
 * @brief   Workflow Global Management Class
 * @details Workflow Global APIs
 */
class WFGlobal
{
public:
	/**
	 * @brief      register default port for one scheme string
	 * @param[in]  scheme           scheme string
	 * @param[in]  port             default port value
	 * @warning    No effect when scheme is "http"/"https"/"redis"/"rediss"/"mysql"/"kafka"
	 */
	static void register_scheme_port(const std::string& scheme, unsigned short port);
	/**
	 * @brief      get default port string for one scheme string
	 * @param[in]  scheme           scheme string
	 * @return     port string const pointer
	 * @retval     NULL             fail, scheme not found
	 * @retval     not NULL         success
	 */
	static const char *get_default_port(const std::string& scheme);
	/**
	 * @brief      get current global settings
	 * @return     current global settings const pointer
	 * @note       returnval never NULL
	 */
	static const struct WFGlobalSettings *get_global_settings();

	static const char *get_error_string(int state, int error);

public:
	// Internal usage only
	static CommScheduler *get_scheduler();
	static DNSCache *get_dns_cache();
	static RouteManager *get_route_manager();
	static SSL_CTX *get_ssl_client_ctx();
	static SSL_CTX *get_ssl_server_ctx();
	static ExecQueue *get_exec_queue(const std::string& queue_name);
	static Executor *get_compute_executor();
	static IOService *get_io_service();
	static ExecQueue *get_dns_queue();
	static Executor *get_dns_executor();
	static WFNameService *get_name_service();
	static void sync_operation_begin();
	static void sync_operation_end();
};
```

### CommScheduler

通用调度器，内部使用`__CommScheduler`单例来管理调度器。

该类为一个全局的调度器，所有任务都通过该类建立异步的任务。

该类也是一个接口类，主要操作都在`class Communicator`中，具体内容可见[[Communicator]]说明。

TODO：具体的作用

### ExecManager

ExecManager主要管理任务执行器以及任务队列。

任务执行器本质是一个线程池，具体说明见[[Executor]]；
任务队列为一个哈希表，保存不同名称的队列。

所有有关计算的任务都由该`manager`进行管理与执行。