## WFThreadTaskFactory

所属文件：factory/WFTaskFactory.h

需要特别注意的是，计算使用的线程池和网络使用的线程池为两个不同的线程池，两个线程归不同的管理者在管理。

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

使用`create_thread_task()`来创建单线程任务，创建任务为[[WFThreadTask#WFThreadTask]]

举例：
```c++
typedef std::vector<std::vector<double>> Matrix;

struct MMInput {
	Matrix a;
	Matrix b;
};

struct MMOutput {
	int error;
	size_t m, n, k;
	Matrix c;
};

bool is_valid_matrix(const Matrix& matrix, size_t& m, size_t& n) {
	m = n = 0;
	if (matrix.size() == 0)
		return true;

	m = matrix.size();
	n = matrix[0].size();
	if (n == 0)
		return false;

	for (const auto& row : matrix)
		if (row.size() != n)
			return false;

	return true;
}

void matrix_multiply(const MMInput *in, MMOutput *out) {
	size_t m1, n1;
	size_t m2, n2;

	if (!is_valid_matrix(in->a, m1, n1) || !is_valid_matrix(in->b, m2, n2)) {
		out->error = EINVAL;
		return;
	}

	if (n1 != m2) {
		out->error = EINVAL;
		return;
	}

	out->error = 0;
	out->m = m1;
	out->n = n2;
	out->k = n1;

	out->c.resize(m1);
	for (size_t i = 0; i < out->m; i++) {
		out->c[i].resize(n2);
		for (size_t j = 0; j < out->n; j++) {
			out->c[i][j] = 0;
			for (size_t k = 0; k < out->k; k++)
				out->c[i][j] += in->a[i][k] * in->b[k][j];
		}
	}
}

using MMTask = WFThreadTask<algorithm::MMInput,
							algorithm::MMOutput>;

using namespace algorithm;

void print_matrix(const Matrix& matrix, size_t m, size_t n) {
	for (size_t i = 0; i < m; i++) 
		for (size_t j = 0; j < n; j++)
			printf("\t%8.2lf", matrix[i][j]);

		printf("\n");
	}
}

void callback(MMTask *task) {
	// 在回调函数中处理计算结果
	auto *input = task->get_input();
	auto *output = task->get_output();

	assert(task->get_state() == WFT_STATE_SUCCESS);

	if (output->error)
		printf("Error: %d %s\n", output->error, strerror(output->error));
	else {
		printf("Matrix A\n");
		print_matrix(input->a, output->m, output->k);
		printf("Matrix B\n");
		print_matrix(input->b, output->k, output->n);
		printf("Matrix A * Matrix B =>\n");
		print_matrix(output->c, output->m, output->n);
	}
}

int main() {
	using MMFactory = WFThreadTaskFactory<MMInput,
										  MMOutput>;
	MMTask *task = MMFactory::create_thread_task("matrix_multiply_task",
												matrix_multiply,
												callback);
	auto *input = task->get_input();

	// 设置输入
	input->a = {{1, 2, 3}, {4, 5, 6}};
	input->b = {{7, 8}, {9, 10}, {11, 12}};

	WFFacilities::WaitGroup wait_group(1);

	Workflow::start_series_work(task, [&wait_group](const SeriesWork *) {
		wait_group.done();
	});

	wait_group.wait();
	return 0;
}
```

### 创建多线程任务

使用`create_multi_thread_task()`来创建多线程任务，得到[[WFThreadTask#WFMultiThreadTask]]

多线程任务就是多个单线程任务的集合，为了方便，在`workflow`也提供了接口。实际运行时，内部相当于创建了多个单线程任务。