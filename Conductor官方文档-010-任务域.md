# Conductor官方文档-010-任务域

任务域帮助支持任务开发。其思想是相同的任务定义可以在不同的领域中实现。域是开发人员控制的任意名称。因此，当工作流启动时，调用方可以在工作流中的所有任务中指定哪些任务需要在特定域中运行，然后使用该域在客户端轮询任务以执行它。

一个例子，如果一个工作流(WF1)有3个任务T1, T2, T3。工作流程被部署并且工作正常，这意味着有T2 worker轮询和执行。如果您修改了T2并在本地运行它，那么无法保证您修改后的T2 worker将获得您所希望的任务，因为它来自一般的T2队列。任务域特性通过按域划分T2队列来解决这个问题，因此当应用程序在特定域中轮询任务T2时，它会得到正确的任务。

当启动工作流时，可以将多个域指定为回退，例如domain1,domain2。Conductor对每个任务的最后轮询时间进行跟踪，因此在本例中，它检查domain1是否有任何活动的worker，然后将任务放入domain1，如果没有，则对顺序为domain2的下一个域进行相同的检查。如果没有工作者处于活动状态，那么任务就是没有域的调度(默认行为)。注意，这种后退类型的域字符串只能在启动工作流时使用，当从客户端轮询时只使用一个域。

### 如何使用任务域

轮询调用现在必须指定域。

> Java

如果您正在使用java，那么一个简单的属性更改将强制WorkflowTaskCoordinator将域传递给轮询者。

```bash
    conductor.worker.T2.domain=mydomain //Task T2 needs to poll for domain "mydomain"
```

> REST调用

```bash
GET /tasks/poll/batch/T2?workerid=myworker&domain=mydomain GET /tasks/poll/T2?workerid=myworker&domain=mydomain
```

> 更改调用启动工作流程

启动工作流时，请确保通过域映射任务

> Java

```java
Map<String, Object> input = new HashMap<>();
input.put("wf_input1", "one”);

Map<String, String> taskToDomain = new HashMap<>();
taskToDomain.put("T2", "mydomain");

// Other options ...
// taskToDomain.put("*", "mydomain") will put all tasks in mydomain
// taskToDomain.put("T2", "mydomain,fallbackDomain") If mydomain has no active workers
//        for T2 then will be put in fallbackDomain. Same can be used with "*" too.

StartWorkflowRequest swr = new StartWorkflowRequest();
swr.withName(“myWorkflow”)
    .withCorrelationId(“corr1”)
    .withVersion(1)
    .withInput(input)
    .withTaskToDomain(taskToDomain);

wfclient.startWorkflow(swr);
```

> REST调用

**POST /workflow**

```bash
{
  "name": "myWorkflow",
  "version": 1,
  "correlatonId": "corr1"
  "input": {
    "wf_input1": "one"
  },
  "taskToDomain": {
    "T2": "mydomain"
  }
}
```

