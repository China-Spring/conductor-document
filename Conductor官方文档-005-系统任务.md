# Conductor官方文档-005-系统任务

> 动态任务

参数：

|名称|描述|
|---|---|
|dynamicTaskNameParam|任务输入中用于计划任务的值的参数名称。例如，如果参数的值是ABC，则调度的下一个任务是“ABC”类型。|

```json
{
  "name": "user_task",
  "taskReferenceName": "t1",
  "inputParameters": {
    "files": "${workflow.input.files}",
    "taskToExecute": "${workflow.input.user_supplied_task}"
  },
  "type": "DYNAMIC",
  "dynamicTaskNameParam": "taskToExecute"
}
```

如果使用输入参数user_supplied_task的值作为**user_task_2**启动工作流，则Conductor将在计划此动态任务时调度**user_task_2**。

> Decision(决策)

决策任务类似于**case...switch**编程语言中的语句。该任务需要3个参数：

参数：

|名称|描述|
|---|---|
|caseValueParam|任务输入中参数的名称，其值将用作开关。|
|decisionCases|可以键入的映射值,**caseValueParam**值是要执行的任务列表。|
|defaultCase|在判定案例中找不到匹配值时要执行的任务列表（默认条件）|

```json
{
  "name": "decide_task",
  "taskReferenceName": "decide1",
  "inputParameters": {
    "case_value_param": "${workflow.input.movieType}"
  },
  "type": "DECISION",
  "caseValueParam": "case_value_param",
  "decisionCases": {
    "Show": [
      {
        "name": "setup_episodes",
        "taskReferenceName": "se1",
        "inputParameters": {
          "movieId": "${workflow.input.movieId}"
        },
        "type": "SIMPLE"
      },
      {
        "name": "generate_episode_artwork",
        "taskReferenceName": "ga",
        "inputParameters": {
          "movieId": "${workflow.input.movieId}"
        },
        "type": "SIMPLE"
      }
    ],
    "Movie": [
      {
        "name": "setup_movie",
        "taskReferenceName": "sm",
        "inputParameters": {
          "movieId": "${workflow.input.movieId}"
        },
        "type": "SIMPLE"
      },
      {
        "name": "generate_movie_artwork",
        "taskReferenceName": "gma",
        "inputParameters": {
          "movieId": "${workflow.input.movieId}"
        },
        "type": "SIMPLE"
      }
    ]
  }
}
```

> Fork

Fork用于调度并行任务集。

参数：

|名称|描述|
|---|---|
|forkTasks|任务列表列表。每个子列表计划并行执行。但是，子列表中的任务是以串行方式安排的。|

```json
{
  "forkTasks": [
    [
      {
        "name": "task11",
        "taskReferenceName": "t11"
      },
      {
        "name": "task12",
        "taskReferenceName": "t12"
      }
    ],
    [
      {
        "name": "task21",
        "taskReferenceName": "t21"
      },
      {
        "name": "task22",
        "taskReferenceName": "t22"
      }
    ]
  ]
}
```

执行时，task11和task21被安排在同一时间执行。

> 动态

动态fork与FORK_JOIN任务相同。除了在运行时使用任务的输入提供要分叉的任务列表。当分叉的任务数量不固定并根据输入而变化时很有用。

|名称|描述|
|---|---|
|dynamicForkTasksParam|包含要并行执行的工作流任务配置列表的参数的名称|
|dynamicForkTasksInputParamName|参数的名称，其值应为带有键的映射，作为分叉任务的引用名称和值作为分叉任务的输入|

```json
{
  "inputParameters": {
     "dynamicTasks": "${taskA.output.dynamicTasksJSON}",
     "dynamicTasksInput": "${taskA.output.dynamicTasksInputJSON}"
  }
  "type": "FORK_JOIN_DYNAMIC",
  "dynamicForkTasksParam": "dynamicTasks",
  "dynamicForkTasksInputParamName": "dynamicTasksInput"
}
```

taskA的输出:

```json
{
  "dynamicTasksInputJSON": {
    "forkedTask1": {
      "width": 100,
      "height": 100,
      "params": {
        "recipe": "jpg"
      }
    },
    "forkedTask2": {
      "width": 200,
      "height": 200,
      "params": {
        "recipe": "jpg"
      }
    }
  },
  "dynamicTasksJSON": [
    {
      "name": "encode_task",
      "taskReferenceName": "forkedTask1",
      "type": "SIMPLE"
    },
    {
      "name": "encode_task",
      "taskReferenceName": "forkedTask2",
      "type": "SIMPLE"
    }
  ]
}
```

执行时，动态fork任务将调度两个类型为“encode_task”的并行任务，引用名称为“forkedTask1”和“forkedTask2”，输入由_dynamicTasksInputJSON_指定

**Join任务必须遵循FORK_JOIN_DYNAMIC
工作流定义必须包含一个Join任务定义，后跟FORK_JOIN_DYNAMIC任务。但是，考虑到任务的动态特性，此Join不需要joinOn参数。在完成之前，连接将等待所有分叉分支完成。
与FORK不同，FORK可以执行并行流，每个fork按顺序执行一系列任务，FORK_JOIN_DYNAMIC仅限于每个fork一个任务。但是，分叉任务可以是子工作流，允许更复杂的执行流。**

> Join

Join任务用于等待fork任务生成的一个或多个任务的完成。

参数:

|名称|描述|
|---|---|
|joinOn|任务引用名称列表，JOIN将等待完成。|

```json
{
    "joinOn": ["taskRef1", "taskRef3"]
}
```

加入任务的输出

Fork任务的输出将是一个JSON对象，其中key是任务引用名称，value是fork任务的输出。

> Sub Workflow

子工作流任务允许在另一个工作流中嵌套工作流。

参数:

|名称|描述|
|---|---|
|subWorkflowParam|任务引用名称列表，JOIN将等待完成。|

```json
{
  "name": "sub_workflow_task",
  "taskReferenceName": "sub1",
  "inputParameters": {
    "requestId": "${workflow.input.requestId}",
    "file": "${encode.output.location}"
  },
  "type": "SUB_WORKFLOW",
  "subWorkflowParam": {
    "name": "deployment_workflow",
    "version": 1
  }
}
```

执行时，**deployment_workflow**使用两个输入参数requestId和file执行 。生成的工作流程完成后，任务标记为已完成。如果子工作流终止或失败，则任务被标记为失败并在配置时重试。

> 等待(Wait)

等待任务被实现为保持在**IN_PROGRESS**状态的任务，除非标记为外部触发器**COMPLETED**或**FAILED**由外部触发器标记。要使用等待任务，请将任务类型设置为**WAIT**

参数：

没有要求。

> 等待任务的外部触发器

任务资源端点可用于将任务的状态更新为终止状态。

Contrib模块提供SQS集成，外部系统可以将消息放入服务器侦听的预配置队列中。当消息到达时，它们被标记为**COMPLETED**或**FAILED**。

> SQS队列

可以使用以下API检索服务器用于更新任务状态的SQS队列：

```bash
GET /queue
```

更新任务状态时，消息需要符合以下规范：

- 消息必须是有效的JSON字符串。
- 消息JSON应包含一个名为key的键externalId，该值是一个包含以下键的JSONified字符串：

    - **workflowId**：工作流程的ID
    - **taskRefName**：应更新的任务引用名称。

- 每个队列代表一个特定的任务状态，并相应地标记任务。例如，发送到**COMPLETED**队列的消息将任务状态标记为**COMPLETED**。
- 任务的输出随消息更新。

有效负载的SQS：

```json
{
  "some_key": "valuex",
  "externalId": "{\"taskRefName\":\"TASK_REFERENCE_NAME\",\"workflowId\":\"WORKFLOW_ID\"}"
}
```

> HTTP

HTTP任务用于通过HTTP调用另一个微服务。

参数:

该任务需要一个输入参数http_request，该参数作为任务输入的一部分，具有以下详细信息：

|名称|描述|
|---|---|
|URI|服务的URI。使用vipAddress或包含服务器地址时可以是部分的。|
|method|HTTP方法。其中一个GET，PUT，POST，DELETE，OPTIONS，HEAD|
|accept|根据服务器的要求接受标头。|
|contentType|内容类型 - 支持的类型是text/plain，text/html和application/json|
|headers|要与请求一起发送的其他http标头的映射。|
|body|请求正文|
|vipAddress|使用基于发现的服务URL。|

HTTP任务输出

|名称|描述|
|---|---|
|response|JSON主体包含响应（如果存在）|
|headers|响应标题|
|statusCode|整数状态代码|

任务输入负载使用vipAddress

```json
{
  "http_request": {
    "vipAddress": "examplevip-prod",
    "uri": "/",
    "method": "GET",
    "accept": "text/plain"
  }
}
```

任务使用绝对URL输入

```json
{
  "http_request": {
    "uri": "http://example.com/",
    "method": "GET",
    "accept": "text/plain"
  }
}
```

该任务被标记为**FAILED**无法完成请求或远程服务器返回非成功的状态代码。

**HTTP任务当前仅支持Content-Type作为application/json，并且能够解析文本以及JSON响应。目前不支持XML输入/输出。但是，如果无法将响应解析为JSON或Text，则将字符串表示形式存储为文本值。**

> 事件(Event)

事件任务提供将事件（消息）发布到Conductor或外部事件系统（如SQS）的功能。事件任务对于为工作流和任务创建基于事件的依赖项非常有用。

参数:

|名称|描述|
|---|---|
|sink|生成的事件的合格名称。例如，导体或sqs：sqs_queue_name|

```json
{
    "sink": 'sqs:example_sqs_queue_name'
}
```

使用Conductor作为接收器生成事件时，事件名称遵循以下结构：**conductor:\<workflow_name\>:\<task_reference_name\>**

对于SQS，请使用队列的**名称**而不是URI。Conductor根据名称查找URI。

**使用SQS时，将ContribsModule添加到部署中。需要使用AWSCredentialsProvider为Conductor配置模块，以便能够使用AWS API。**

支持的接收器

- Conductor
- SQS

事件任务输入

给予事件任务的输入可作为有效负载用于已发布的消息。例如，如果消息被放入SQS队列（接收器是sqs），则消息有效负载将是任务的输入。

事件任务输出

**event_produced** 生成的事件的名称。

