# Conductor官方文档-012-API

> Task & Workflow Metadata

|接口路径|描述|输入|
|---|---|---|
|GET /metadata/taskdefs|获取所有任务定义|N/A|
|GET /metadata/taskdefs/{taskType}|检索任务定义|任务名称|
|POST /metadata/taskdefs|注册新的任务定义|任务定义列表|
|PUT /metadata/taskdefs|更新任务定义|一个任务定义|
|DELETE /metadata/taskdefs/{taskType}|删除任务定义|任务名称|
|GET /metadata/workflow|获取所有工作流程定义|N/A|
|POST /metadata/workflow|注册新工作流程|工作流定义|
|PUT /metadata/workflow|注册/更新新工作流程|工作流定义列表|
|GET /metadata/workflow/{name}?version=|获取工作流程定义|工作流程名称，版本（可选）|

> 启动工作流程输入

```bash
POST /workflow/{name}?version=&correlationId=
{
   //JSON payload for workflow
}
```

|参数|描述|
|---|---|
|version|可选的。如果未指定，则使用最新版本的工作流程|
|correlationID	|用户提供的Id，可用于检索工作流程|

- 输入

JSON Payload启动工作流程。强制性。如果工作流程不期望任何输入必须传递空的JSON，如{}

- 输出

工作流程的ID（GUID）

> 使用输入和任务域

```bash
POST /workflow
{
   //JSON payload for Start workflow request
}
```

> 启动工作流请求

用于启动工作流请求的JSON

```json
{
  "name": "myWorkflow", // Name of the workflow
  "version": 1, // Version
  “correlatond”: “corr1” // correlation Id
  "input": {
    // Input map. 
  },
  "taskToDomain": {
    // Task to domain map
  }
}
```

- 输出

工作流程的ID（GUID）

> 检索工作流程

|接口地址|描述|
|---|---|
|GET /workflow/{workflowId}?includeTasks=true\|false|按工作流ID获取工作流状态。如果设置了includeTasks，则还包括已执行和计划的所有任务。|
|GET /workflow/running/{name}|获取给定类型的所有正在运行的工作流程|
|GET /workflow/running/{name}/correlated/{correlationId}?includeClosed=true\|false&includeTasks=true\|false|获取按相关ID过滤的所有正在运行的工作流程。如果设置了includeClosed，还包括已完成运行的工作流。|
|GET /workflow/search|搜索工作流程。|

> 搜索工作流程

Conductor使用Elasticsearch来索引工作流程执行，并由搜索API使用。

**GET /workflow/search?start=&size=&sort=&freeText=&query=**

|参数|描述|
|---|---|
|start|页码。默认为0|
|size|要返回的结果数|
|sort|排序。格式为：ASC:\<fieldname\>或DESC:\<fieldname\>按字段按升序或降序排序|
|freeText|Elasticsearch支持查询。例如workflowType：“name_of_workflow”|
|query|SQL喜欢where子句。例如workflowType ='name_of_workflow'。如果提供了freeText，则可选。|

- 输出

```json
{
  "totalHits": 0,
  "results": [
    {
      "workflowType": "string",
      "version": 0,
      "workflowId": "string",
      "correlationId": "string",
      "startTime": "string",
      "updateTime": "string",
      "endTime": "string",
      "status": "RUNNING",
      "input": "string",
      "output": "string",
      "reasonForIncompletion": "string",
      "executionTime": 0,
      "event": "string"
    }
  ]
}
```

> 管理工作流程

![](http://image.cdn.ttxit.com/15380402999664.jpg)

> 重新运行

从特定任务重新运行已完成的工作流程。

**POST /workflow/{workflowId}/rerun**

```json
{
  "reRunFromWorkflowId": "string",
  "workflowInput": {},
  "reRunFromTaskId": "string",
  "taskInput": {}
}
```

> 跳过任务

跳过正在运行的工作流中的任务执行(指定为taskReferenceName参数)并继续前进。可选地按照负载中指定的方式更新任务的输入和输出。**PUT /workflow/{workflowId}/skiptask/{taskReferenceName}?workflowId=&taskReferenceName=**

```json
{
  "taskInput": {},
  "taskOutput": {}
}
```

> 管理任务

![](http://image.cdn.ttxit.com/15380403966217.jpg)

> 轮询，确认和更新任务

这些是用于轮询任务，发送确认（轮询后）并最终由worler更新任务结果的关键端点。

![](http://image.cdn.ttxit.com/15380404520887.jpg)

> 用于更新任务结果的模式

```json
{
    "workflowInstanceId": "Workflow Instance Id",
    "taskId": "ID of the task to be updated",
    "reasonForIncompletion" : "If failed, reason for failure",
    "callbackAfterSeconds": 0,
    "status": "IN_PROGRESS|FAILED|COMPLETED",
    "outputData": {
        //JSON document representing Task execution output     
    }

}
```

