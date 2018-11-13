# Conductor官方文档-007-Kitchensink实例

Kitchensink工作流示例，演示了所有模式构造的用法的。

> 定义

```json
{
  "name": "kitchensink",
  "description": "kitchensink workflow",
  "version": 1,
  "tasks": [
    {
      "name": "task_1",
      "taskReferenceName": "task_1",
      "inputParameters": {
        "mod": "${workflow.input.mod}",
        "oddEven": "${workflow.input.oddEven}"
      },
      "type": "SIMPLE"
    },
    {
      "name": "event_task",
      "taskReferenceName": "event_0",
      "inputParameters": {
        "mod": "${workflow.input.mod}",
        "oddEven": "${workflow.input.oddEven}"
      },
      "type": "EVENT",
      "sink": "conductor"
    },
    {
      "name": "dyntask",
      "taskReferenceName": "task_2",
      "inputParameters": {
        "taskToExecute": "${workflow.input.task2Name}"
      },
      "type": "DYNAMIC",
      "dynamicTaskNameParam": "taskToExecute"
    },
    {
      "name": "oddEvenDecision",
      "taskReferenceName": "oddEvenDecision",
      "inputParameters": {
        "oddEven": "${task_2.output.oddEven}"
      },
      "type": "DECISION",
      "caseValueParam": "oddEven",
      "decisionCases": {
        "0": [
          {
            "name": "task_4",
            "taskReferenceName": "task_4",
            "inputParameters": {
              "mod": "${task_2.output.mod}",
              "oddEven": "${task_2.output.oddEven}"
            },
            "type": "SIMPLE"
          },
          {
            "name": "dynamic_fanout",
            "taskReferenceName": "fanout1",
            "inputParameters": {
              "dynamicTasks": "${task_4.output.dynamicTasks}",
              "input": "${task_4.output.inputs}"
            },
            "type": "FORK_JOIN_DYNAMIC",
            "dynamicForkTasksParam": "dynamicTasks",
            "dynamicForkTasksInputParamName": "input"
          },
          {
            "name": "dynamic_join",
            "taskReferenceName": "join1",
            "type": "JOIN"
          }
        ],
        "1": [
          {
            "name": "fork_join",
            "taskReferenceName": "forkx",
            "type": "FORK_JOIN",
            "forkTasks": [
              [
                {
                  "name": "task_10",
                  "taskReferenceName": "task_10",
                  "type": "SIMPLE"
                },
                {
                  "name": "sub_workflow_x",
                  "taskReferenceName": "wf3",
                  "inputParameters": {
                    "mod": "${task_1.output.mod}",
                    "oddEven": "${task_1.output.oddEven}"
                  },
                  "type": "SUB_WORKFLOW",
                  "subWorkflowParam": {
                    "name": "sub_flow_1",
                    "version": 1
                  }
                }
              ],
              [
                {
                  "name": "task_11",
                  "taskReferenceName": "task_11",
                  "type": "SIMPLE"
                },
                {
                  "name": "sub_workflow_x",
                  "taskReferenceName": "wf4",
                  "inputParameters": {
                    "mod": "${task_1.output.mod}",
                    "oddEven": "${task_1.output.oddEven}"
                  },
                  "type": "SUB_WORKFLOW",
                  "subWorkflowParam": {
                    "name": "sub_flow_1",
                    "version": 1
                  }
                }
              ]
            ]
          },
          {
            "name": "join",
            "taskReferenceName": "join2",
            "type": "JOIN",
            "joinOn": [
              "wf3",
              "wf4"
            ]
          }
        ]
      }
    },
    {
      "name": "search_elasticsearch",
      "taskReferenceName": "get_es_1",
      "inputParameters": {
        "http_request": {
          "uri": "http://localhost:9200/conductor/_search?size=10",
          "method": "GET"
        }
      },
      "type": "HTTP"
    },
    {
      "name": "task_30",
      "taskReferenceName": "task_30",
      "inputParameters": {
        "statuses": "${get_es_1.output..status}",
        "workflowIds": "${get_es_1.output..workflowId}"
      },
      "type": "SIMPLE"
    }
  ],
  "outputParameters": {
    "statues": "${get_es_1.output..status}",
    "workflowIds": "${get_es_1.output..workflowId}"
  },
  "schemaVersion": 2
}
```

> Visual Flow

![](http://image.cdn.ttxit.com/15380371644714.png)

> 运行Kitchensink工作流程

- 按照[此处的](http://www.springall.com.cn/article/119)说明启动服务器。**-DloadSample=true**启动服务器时使用java系统属性。这将创建一个kitchennsink工作流程，相关的任务定义和启动kitchennsink工作流程的实例。
- 工作流程启动后，第一个任务仍处于该**SCHEDULED**状态。这是因为目前没有工人正在轮询该任务。
- 我们将直接使用REST端点轮询任务并更新状态。

> 开始执行工作流

通过发布以下内容开始执行kitchennsink工作流程：

```json
curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/plain' 'http://localhost:8080/api/workflow/kitchensink' -d '
{
    "task2Name": "task_5" 
}
'
```

响应是标识工作流实例标识的文本字符串。

> 第一项任务

```bash
curl http://localhost:8080/api/tasks/poll/task_1
```

响应应该类似于：

```json
{
    "taskType": "task_1",
    "status": "IN_PROGRESS",
    "inputData": {
        "mod": null,
        "oddEven": null
    },
    "referenceTaskName": "task_1",
    "retryCount": 0,
    "seq": 1,
    "pollCount": 1,
    "taskDefName": "task_1",
    "scheduledTime": 1486580932471,
    "startTime": 1486580933869,
    "endTime": 0,
    "updateTime": 1486580933902,
    "startDelayInSeconds": 0,
    "retried": false,
    "callbackFromWorker": true,
    "responseTimeoutSeconds": 3600,
    "workflowInstanceId": "b0d1a935-3d74-46fd-92b2-0ca1e388659f",
    "taskId": "b9eea7dd-3fbd-46b9-a9ff-b00279459476",
    "callbackAfterSeconds": 0,
    "polledTime": 1486580933902,
    "queueWaitTime": 1398
}
```

> 更新任务状态

- 请注意轮询响应的值**taskId**和**workflowInstanceId**字段
- 更新任务的**COMPLETED**状态，如下所示：

```json
curl -H 'Content-Type:application/json' -H 'Accept:application/json' -X POST http://localhost:8080/api/tasks/ -d '
{
    "taskId": "b9eea7dd-3fbd-46b9-a9ff-b00279459476",
    "workflowInstanceId": "b0d1a935-3d74-46fd-92b2-0ca1e388659f",
    "status": "COMPLETED",
    "output": {
        "mod": 5,
        "taskToExecute": "task_1",
        "oddEven": 0,
        "dynamicTasks": [
            {
                "name": "task_1",
                "taskReferenceName": "task_1_1",
                "type": "SIMPLE"
            },
            {
                "name": "sub_workflow_4",
                "taskReferenceName": "wf_dyn",
                "type": "SUB_WORKFLOW",
                "subWorkflowParam": {
                    "name": "sub_flow_1"
                }
            }
        ],
        "inputs": {
            "task_1_1": {},
            "wf_dyn": {}
        }
    }
}'
```

这将把task_1标记为已完成，并安排task_5为下一个任务。

对随后计划的任务重复相同的过程，直到完成。

