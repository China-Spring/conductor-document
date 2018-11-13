# Conductor官方文档-009-Conductor Task Workers

远程Workers执行的Task任务通过HTTP端点进行通信，以轮询任务并更新执行状态。

Conductor提供了一个框架来轮询任务，管理执行线程并将执行状态更新回服务器。该框架提供Java和Python库。可以使用HTTP端点进行任务管理来添加其他语言支持。

> java

- 实现[Worker](https://github.com/Netflix/conductor/blob/dev/client/src/main/java/com/netflix/conductor/client/worker/Worker.java)接口以实现任务。
- 使用[WorkflowTaskCoordinator](https://github.com/Netflix/conductor/blob/dev/client/src/main/java/com/netflix/conductor/client/task/WorkflowTaskCoordinator.java)注册worker并初始化轮询循环。

    - [Sample Worker Implementation](https://github.com/Netflix/conductor/blob/dev/client/src/test/java/com/netflix/conductor/client/sample/SampleWorker.java)
    - [Example](https://github.com/Netflix/conductor/blob/dev/client/src/test/java/com/netflix/conductor/client/sample/Main.java)

> WorkflowTaskCoordinator

管理Task worker线程池和服务器通信（poll，task update和ack）。

> Worker

|属性|描述|
|---|---|
|paused|布尔值。如果设置为true，则工作程序将停止轮询。|
|pollCount|轮询的任务数量。用于批量轮询。每个任务都在一个单独的线程中执行。|
|longPollTimeout|长时间轮询到Conductor服务器执行任务的时间（以毫秒为单位）|

可以通过Worker实现或通过在JVM中设置以下系统属性来设置这些属性：

|属性|描述|
|---|---|
|conductor.worker.\<property\>|适用于JVM中的所有工作程序|
|conductor.worker.\<taskDefName\>.\<property\>|适用于指定的工作者。覆盖全局属性。|

> Python

[https://github.com/Netflix/conductor/tree/dev/client/python](https://github.com/Netflix/conductor/tree/dev/client/python)

按照自述文件中的说明进行操作，或者查看[kitchensink_workers.py](https://github.com/Netflix/conductor/blob/dev/client/python/kitchensink_workers.py)

