# Conductor官方文档-004-元数据定义

> 任务定义

Conductor维护着一个worker task注册表。在工作流程中使用之前必须注册任务类型。

```json
{
  "name": "encode_task",
  "retryCount": 3,
  "timeoutSeconds": 1200,
  "inputKeys": [
    "sourceRequestId",
    "qcElementType"
  ],
  "outputKeys": [
    "state",
    "skipped",
    "result"
  ],
  "timeoutPolicy": "TIME_OUT_WF",
  "retryLogic": "FIXED",
  "retryDelaySeconds": 600,
  "responseTimeoutSeconds": 3600
}
```

|字段|描述|说明|
|---|---|---|
|name|任务类型|唯一|
|retryCount|任务标记为失败时尝试重试的次数|-|
|retryLogic|重试机制|请参见下面的可能值|
|timeoutSeconds|以毫秒为单位的时间，在此之后，如果在转换到`IN_PROGRESS`状态后未完成任务，则将任务标记为TIMED_OUT|如果设置为0，则不会超时|
|timeoutPolicy|任务的超时策略|请参见下面的可能值|
|responseTimeoutSeconds|如果大于0，则在此时间之后未更新状态时，将重新安排任务。当work轮询任务但由于错误/网络故障而无法完成时很有用。|-|
|outputKeys|任务输出的键集。用于记录任务的输出|-|

- 重试逻辑

    - FIXED: 在**retryDelaySeconds**之后重新安排任务
    - EXPONENTIAL_BACKOFF: **retryDelaySeconds  * attempNo**重试时间

- 超时政策

    - RETRY: 再次重试该任务
    - TIME_OUT_WF: 工作流程标记为TIMED_OUT并终止
    - ALERT_ONLY: 注册计数器（task_timeout）

> 工作流定义

使用基于JSON的DSL定义工作流。

```json
{
  "name": "encode_and_deploy",
  "description": "Encodes a file and deploys to CDN",
  "version": 1,
  "tasks": [
    {
      "name": "encode",
      "taskReferenceName": "encode",
      "type": "SIMPLE",
      "inputParameters": {
        "fileLocation": "${workflow.input.fileLocation}"
      }
    },
    {
      "name": "deploy",
      "taskReferenceName": "d1",
      "type": "SIMPLE",
      "inputParameters": {
        "fileLocation": "${encode.output.encodeLocation}"
      }

    }
  ],
  "outputParameters": {
    "cdn_url": "${d1.output.location}"
  },
  "schemaVersion": 2
}
```

|字段|描述|说明|
|---|---|---|
|name|工作流程的名称|-|
|description|工作流程的描述|-|
|version|用于标识架构版本的数字字段。使用递增数字|启动工作流程执行时，如果未指定，则使用具有最高版本的定义|
|tasks|一系列任务定义，如下所述。|-|
|outputParameters|用于生成工作流输出的JSON模板|如果未指定，则将输出定义为**上次**执行的任务的输出|
|inputParameters|输入参数列表。用于记录工作流程所需的输入|可选的|

> 工作流程中的任务

**tasks**工作流中的属性定义要按该顺序执行的任务数组。以下是每项任务所需的强制性最低参数：

|字段|描述|说明|
|---|---|---|
|name|任务名称。在开始工作流程之前，必须使用Conductor注册为任务类型|-|
|taskReferenceName|别名用于在工作流程中引用任务。必须是独一无二的。|-|
|type|任务类型。SIMPLE用于远程work或其中一个系统任务类型执行的任务|-|
|description|任务描述|可选的|
|optional|设置为true时 - 即使任务失败，工作流也会继续。任务的状态反映为**COMPLETED_WITH_ERRORS**|默认为**false**|
|inputParameters|JSON模板，用于定义任务的输入|有关详细信息，请参见“连接输入和输出”|

除了这些参数，需要进行具体的任务类型附加参数记录[Conductor官方文档-第005章-系统任务](mweblib://15330885338375)

> 连接输入和输出

当触发新的执行时，客户端会为工作流提供输入。工作流输入是通过**${workflow.input...}**表达式提供的JSON有效负载。

基于**inputParameters**工作流定义中配置的模板，为工作流中的每个任务提供输入。 **inputParameters**是一个JSON片段，其值包含用于在执行期间映射工作流的输入或输出或其他任务的值的参数。

映射值的语法遵循以下模式：

`${SOURCE.input/output.JSONPath}`

|-|-|
|---|---|
|SOURCE|可以是任何任务的“工作流程”或引用名称|
|input/output|指源的输入或输出|
|JSONPath|JSON路径表达式从源的输入/输出中提取JSON片段|

**Conductor支持[JSONPath规范](http://goessner.net/articles/JsonPath/)并从此处[使用Java实现](https://github.com/jayway/JsonPath)。**

考虑一个任务，其输入配置为使用来自工作流的输入/输出参数和名为**loc_task**的任务。

```json
{
  "inputParameters": {
    "movieId": "${workflow.input.movieId}",
    "url": "${workflow.input.fileLocation}",
    "lang": "${loc_task.output.languages[0]}",
    "http_request": {
      "method": "POST",
      "url": "http://example.com/${loc_task.output.fileId}/encode",
      "body": {
        "recipe": "${workflow.input.recipe}",
        "params": {
          "width": 100,
          "height": 100
        }
      },
      "headers": {
        "Accept": "application/json",
        "Content-Type": "application/json"
      }
    }
  }
}
```

请将以下内容视为工作流输入

```json
{
  "movieId": "movie_123",
  "fileLocation":"s3://moviebucket/file123",
  "recipe":"png"
}
```

并且loc_task的输出如下;

```json
{
  "fileId": "file_xxx_yyy_zzz",
  "languages": ["en","ja","es"]
}
```

在安排任务时，Conductor将合并工作流输入和loc_task输出中的值，并按如下方式创建任务输入：

```json
{
  "movieId": "movie_123",
  "url": "s3://moviebucket/file123",
  "lang": "en",
  "http_request": {
    "method": "POST",
    "url": "http://example.com/file_xxx_yyy_zzz/encode",
    "body": {
      "recipe": "png",
      "params": {
        "width": 100,
        "height": 100
      }
    },
    "headers": {
        "Accept": "application/json",
        "Content-Type": "application/json"
    }
  }
}
```

