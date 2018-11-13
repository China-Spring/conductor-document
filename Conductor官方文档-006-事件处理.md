# Conductor官方文档-006-事件处理

Conductor中的事件提供了工作流之间的松散耦合以及对从外部系统生成和使用事件的支持。

这包括：

1. 能够在外部系统（如SQS或内部导体）中生成事件（消息）。
2. 在发生与提供的条件匹配的特定事件时启动工作流。

Conductor提供SUB_WORKFLOW任务，可用于在父工作流中嵌入工作流。事件支持提供了类似的功能，无需显式添加依赖项，并提供了**即发即弃**样式集成。

> 活动任务

事件任务提供将事件（消息）发布到Conductor或外部事件系统（如SQS）的功能。事件任务对于为工作流和任务创建基于事件的依赖项非常有用。

有关文档，请参阅[事件任务](http://www.springall.com.cn/article/36)

> 事件处理程序

事件处理程序是已注册的侦听器，在发生匹配事件时执行操作。支持的操作是：

1. 启动工作流程
2. 失败任务
3. 完成任务

可以将事件处理程序配置为侦听指挥事件或SQS等外部事件。

> Configuration

事件处理程序通过**/event/** API配置。

结构体：

```json
{
  "name" : "descriptive unique name",
  "event": "event_type:event_location",
  "condition": "boolean condition",
  "actions": ["see examples below"]
}
```

条件:

条件是必须计算为布尔值的表达式。支持类似Javascript的语法，可用于根据负载评估条件。只有当条件求值为true时才执行操作。

给出消息中的以下有效负载：

```json
{
    "fileType": "AUDIO",
    "version": 3,
    "metadata": {
       length: 300,
       codec: "aac"
    }
}
```

|表达式|结果|
|---|---|
|$.version > 1	|true|
|$.version > 10|false|
|$.metadata.length == 300|true|

### 操作 

> 启动工作流程

```json
{
    "action": "start_workflow",
    "start_workflow": {
        "name": "WORKFLOW_NAME",
        "version": <optional>
        "input": {
            "param1": "${param1}" 
        }
    }
}
```

> 完成任务 *

```json
{
    "action": "complete_task",
    "complete_task": {
      "workflowId": "${source.externalId.workflowId}",
      "taskRefName": "task_1",
      "output": {
        "response": "${source.result}"
      }
    },
    "expandInlineJSON": true
}
```

> 失败任务 *

```json
{
    "action": "fail_task",
    "fail_task": {
      "workflowId": "${source.externalId.workflowId}",
      "taskRefName": "task_1",
      "output": {
        "response": "${source.result}"
      }
    },
    "expandInlineJSON": true
}
```

完成/失败任务时启动工作流和输出的输入遵循用于连接工作流输入的[相同表达式](http://www.springall.com.cn/article/35)。

**在有效负载中扩展字符串化的JSON元素**

**expandInlineJSON** property，当设置为true时，会将有效内容中的内联字符串化JSON元素扩展为JSON文档，并将字符串值替换为JSON文档。
此功能允许此类元素与JSON路径表达式一起使用。

> 扩展

提供[EventQueueProvider](https://github.com/Netflix/conductor/blob/master/core/src/main/java/com/netflix/conductor/core/events/EventQueueProvider.java)的实现。

SQS队列提供程序： [SQSEventQueueProvider.java](https://github.com/Netflix/conductor/blob/master/contribs/src/main/java/com/netflix/conductor/core/events/sqs/SQSEventQueueProvider.java)


