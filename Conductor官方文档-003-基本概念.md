# Conductor官方文档-003-基本概念

> 工作流定义

工作流是基于JSON定义的DSL，并且包含作为工作流的一部分执行的一组任务。这些任务要么是控制任务(fork，conditional etc等)，要么是应用程序任务（例如编码文件）。[更多细节](https://netflix.github.io/conductor/metadata)

> 任务定义

- 所有任务都需要在活动工作流程使用之前进行注册。
- 任务可以在多个工作流程中重复使用。任务分为两类：
    
    - 系统任务
    - Worker任务

> 系统任务

系统任务在Conductor服务器的JVM内执行，并由Conductor管理，以实现其可执行性和可扩展性。

| 名称  | 目的 |
| --- | --- |
| [DYNAMIC](https://netflix.github.io/conductor/metadata/systask/#dynamic-task) | 基于任务的输入表达式派生的工作任务，而不是静态蓝图的一部分 |
|[DECIDE](https://netflix.github.io/conductor/metadata/systask/#decision)|决策任务 - 实现案例......开关样式分叉|
|[FORK](https://netflix.github.io/conductor/metadata/systask/#fork)|分叉一组并行的任务。计划每个集合并行执行|
|[FORK_JOIN_DYNAMIC](https://netflix.github.io/conductor/metadata/systask/#dynamic-fork)|与FORK类似，但FORK_JOIN_DYNAMIC不是在并行执行蓝图中定义的任务集，而是根据此任务的输入表达式生成并行任务|
|[JOIN](https://netflix.github.io/conductor/metadata/systask/#join)|补充FORK和FORK_JOIN_DYNAMIC。用于合并一个或多个并行分支*|
|[SUB_WORKFLOW](https://netflix.github.io/conductor/metadata/systask/#sub-workflow)|将另一个工作流嵌套为子工作流任务。在执行时，它实例化子工作流并等待它完成|
|[EVENT](https://netflix.github.io/conductor/metadata/systask/#event)|在支持的事件系统中生成事件（例如，Conductor，SQS）|

Conductor提供了一个API来创建用户定义的任务，这些任务在与引擎相同的JVM中执行。详细信息请参见[WorkflowSystemTask](https://github.com/Netflix/conductor/blob/dev/core/src/main/java/com/netflix/conductor/core/execution/tasks/WorkflowSystemTask.java)接口。

> Worker任务

Worker任务由应用程序实现，并在与Conductor不同的环境中运行。Worker任务可以用任何语言实现。这些任务通过REST API端点与Conductor服务器通信，以轮询任务并在执行后更新其状态。

Worker任务由蓝图中的任务类型**SIMPLE**标识。

> 工作流任务的生命周期

![](http://image.cdn.ttxit.com/15320677848727.png)

