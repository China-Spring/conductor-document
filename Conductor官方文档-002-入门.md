# Conductor官方文档-002-入门

> 高级架构

![](http://image.cdn.ttxit.com/15319682282531.png)

API和存储层是可插拔的，可以与不同的后端和队列服务提供商一起工作。

### 安装和运行

**有关在生产中安装和运行Conductor服务的详细配置指南，请访问[Conductor Server](https://netflix.github.io/conductor/server)文档。**

> 在内存中运行服务

按照以下步骤快速启动由内存数据库支持的本地Conductor实例，其中包含一个简单的kitchen sink workflow，该工作流程演示了Conductor的所有功能。

**警告：内存服务器用于快速演示目的，不会将数据存储在磁盘上。一旦服务器关闭，所有数据都将丢失。**

> 从github克隆源代码

```bash
git clone git@github.com:Netflix/conductor.git
```

> 启动本地服务

```bash
cd server
../gradlew server
# wait for the server to come online
```

可以在[http://localhost:8080](http://localhost:8080/)访问Swagger API

> 启动UI服务器

```bash
cd ui
gulp watch
```

> 或者使用[docker-compose](https://netflix.github.io/conductor/docker/docker-compose.yaml)启动所有服务

```bash
cd docker
docker-compose up
```

如果您在本地运行它，请访问[http://localhost:3000/](http://localhost:3000/)启动UI，或者如果您使用docker-compose运行它，则在[http://localhost:5000/](http://localhost:5000/)启动UI。

**注意：默认情况下，服务器将加载示例kitchen sink workflow定义。详情请见[此处](https://netflix.github.io/conductor/metadata/kitchensink/)。**

### 运行模型

Conductor遵循基于RPC的通信模型，其中工作节点在与服务器不同的机器上运行。工作节点通过基于HTTP的端点与服务器通信，并使用轮询模型来管理工作队列。

![](http://image.cdn.ttxit.com/15319688174219.png)

* 工作节点是远程系统，通过HTTP（或任何支持的RPC机制）与主服务器进行通信。
* 任务队列用于为工作节点安排任务。我们在内部使用[dyno-queues](https://github.com/Netflix/dyno-queues)它可以很容易地与SQS或类似的pub-sub机制交换。
* conductor-redis-persistence模块使用[Dynomite](https://github.com/Netflix/dynomite)存储状态和元数据以及[Elasticsearch](https://www.elastic.co/)来索引后端。
* 请参阅扩展后端部分，以实现对不同数据库的存储和索引支持。

### 高级步骤

注册和执行新工作流程所需的步骤：

1. 定义工作流使用的任务。
2. 创建工作流程
3. 创建定期轮询计划任务的任务工作程序

> 触发执行工作流程

```bash
POST /workflow/{name}
{
    ... //json payload as workflow input
}
```

> 轮询任务

```bash
GET /tasks/poll/batch/{taskType}
```

> 更新任务状态

```bash
POST /tasks
{
    "outputData": {
        "encodeResult":"success",
        "location": "http://cdn.example.com/file/location.png"
        //any task specific output
     },
     "status": "COMPLETED"
}
```

