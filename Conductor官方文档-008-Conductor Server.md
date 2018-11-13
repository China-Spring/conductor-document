# Conductor官方文档-008-Conductor Server

> 安装要求

- 数据库：[Dynomite](https://github.com/Netflix/dynomite)
- 索引后端：[Elasticsearch 2.x.](https://www.elastic.co/)
- Servlet容器：运行JDK 1.8或更高版本的Tomcat，Jetty或类似产品

您可以通过3种方式安装Conductor：

> Source源码构建

要从源代码构建，请使用**gradle build**命令从github和构建服务器模块中检出代码。如果未安装gradle，则可以**./gradlew build**从项目根目录运行该命令。这会在文件夹**./server/build/libs/**中生成**conductor-server-all-VERSION.jar**

jar可以使用以下方式执行：

```bash
java -jar conductor-server-VERSION-all.jar
```

> 从jcenter或maven central下载预构建的二进制文件

|group|artifact|version|
|---|---|---|
|com.netflix.conductor|conductor-server-all|1.6.+|

> 使用预配置的Docker镜像

要为conductor server和ui构建docker映像，运行以下命令:

```bash
cd docker
docker-compose build
```

构建docker镜像后，运行以下命令以启动容器：

```bash
docker-compose up
```

这将创建一个docker容器网络，其中包含以下图像：conductor:server, conductor:ui, [elasticsearch:2.4](https://hub.docker.com/_/elasticsearch/)和dynomite。

要查看UI，请浏览器访问[localhost:5000](localhost:5000)，要查看Swagger文档，请浏览器访问[localhost:8080](localhost:8080)。

### Configuration

Conductor服务器使用基于属性文件的配置。属性文件作为命令行参数传递给Main类。

```bash
java -jar conductor-server-all-VERSION.jar [PATH TO PROPERTY FILE] [log4j.properties file path]
```

log4j.properties文件路径是可选的，允许更好地控制日志记录（默认为控制台中的INFO级别日志记录）。

> 配置参数

```java
# Database persistence model.  Possible values are memory, redis, redis_cluster and dynomite.
# If omitted, the persistence used is memory
#
# memory : The data is stored in memory and lost when the server dies.  Useful for testing or demo
# redis : non-Dynomite based redis instance
# redis_cluster: AWS Elasticache Redis (cluster mode enabled).See [http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Clusters.Create.CON.RedisCluster.html]
# dynomite : Dynomite cluster.  Use this for HA configuration.
db=dynomite

# Dynomite Cluster details.
# format is host:port:rack separated by semicolon
# for AWS Elasticache Redis (cluster mode enabled) the format is configuration_endpoint:port:us-east-1e. The region in this case does not matter
workflow.dynomite.cluster.hosts=host1:8102:us-east-1c;host2:8102:us-east-1d;host3:8102:us-east-1e

# Dynomite cluster name
workflow.dynomite.cluster.name=dyno_cluster_name

# Maximum connections to redis/dynomite
workflow.dynomite.connection.maxConnsPerHost=31

# Namespace for the keys stored in Dynomite/Redis
workflow.namespace.prefix=conductor

# Namespace prefix for the dyno queues
workflow.namespace.queue.prefix=conductor_queues

# No. of threads allocated to dyno-queues (optional)
queues.dynomite.threads=10

# Non-quorum port used to connect to local redis.  Used by dyno-queues.
# When using redis directly, set this to the same port as redis server
# For Dynomite, this is 22122 by default or the local redis-server port used by Dynomite.
queues.dynomite.nonQuorum.port=22122

# Transport address to elasticsearch
workflow.elasticsearch.url=localhost:9300

# Name of the elasticsearch cluster
workflow.elasticsearch.index.name=conductor

# Additional modules (optional)
conductor.additional.modules=class_extending_com.google.inject.AbstractModule
```

> 高可用性配置

Conductor服务器是无状态的，可以部署在多个服务器上，以满足规模和可用性需求。通过扩展[Dynomite](https://github.com/Netflix/dynomite)集群以及用于队列的[dyno-queue](https://github.com/Netflix/dyno-queues)来实现服务器的可伸缩性。

客户端通过HTTP负载均衡器或使用Discovery（NetflixOSS）连接到服务器。

> 使用独立Redis/ElastiCache

Conductor服务器可以与Standlone Redis或ElastiCache服务器一起使用。要配置服务器，请更改配置以使用以下内容：

```java
db=redis

# For AWS Elasticache Redis (cluster mode enabled) the format is configuration_endpoint:port:us-east-1e. 
# The region in this case does not matter
workflow.dynomite.cluster.hosts=server_address:server_port:us-east-1e
workflow.dynomite.connection.maxConnsPerHost=31

queues.dynomite.nonQuorum.port=server_port
```


