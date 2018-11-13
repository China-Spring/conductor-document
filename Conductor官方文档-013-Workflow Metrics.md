# Conductor官方文档-013-Workflow Metrics

> Worker Metrics

Conductor使用[spectator](https://github.com/Netflix/spectator)来收集指标。

|名称|目的|标签|
|---|---|---|
|workflow_server_error|服务器端错误发生的速率|methodName|
|workflow_failure	|失败的工作流程的计数器|workflowName, status|
|workflow_start_error|无法启动工作流程的计数器|workflowName|
|workflow_running	|运行工作流程|workflowName,version|
|task_queue_wait|队列中任务所花费的时间|taskType|
|task_execution|执行任务所花费的时间|taskType, includeRetries, status|
|task_poll|轮询任务所花费的时间|taskType|
|task_queue_depth|待处理任务队列深度|taskType|
|task_timeout|计时器超时任务	|taskType|

> Worker Metrics

使用Java时，将发布以下度量标准：

|名称|目的|标签|
|---|---|---|
|task_execution_queue_full|记录执行队列的计数器已经饱和|taskType|
|task_poll_error|轮询任务队列时出现客户端错误|taskType, includeRetries, status|
|task_execute_error|执行错误|taskType|
|task_ack_failed|任务确认失败|taskType|
|task_ack_error|任务确认遇到异常|taskType|
|task_update_error|任务状态无法更新回服务器|taskType|
|task_poll_counter|执行任务总数|taskType|
|task_poll_time|批量执行任务时间|taskType|
|task_execute_time|执行任务时间|taskType|