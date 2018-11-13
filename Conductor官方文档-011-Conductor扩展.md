# Conductor官方文档-011-Conductor扩展

Conductor提供一个可插入的后端。当前的实现使用Dynomite。

每个后端需要实现4个接口：

```java
//Store for workflow and task definitions
com.netflix.conductor.dao.MetadataDAO
```

```java
//Store for workflow executions
com.netflix.conductor.dao.ExecutionDAO
```

```java
//Index for workflow executions
com.netflix.conductor.dao.IndexDAO
```

```java
//Queue provider for tasks
com.netflix.conductor.dao.QueueDAO
```

可以为每种实现混合并匹配不同的实现。例如用于排队的SQS和用于其他人的关系存储。

> 系统任务

要创建系统任务，请按照以下步骤操作：

- 扩展**com.netflix.conductor.core.execution.tasks.WorkflowSystemTask**
- 将新类实例化为启动的一部分(动态单例)

