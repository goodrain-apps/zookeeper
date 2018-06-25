# 应用-kafka

## 什么是kafka
Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。该项目的目标是为处理实时数据提供一个统一、高吞吐、低延迟的平台。其持久化层本质上是一个“按照分布式事务日志架构的大规模发布/订阅消息队列”，这使它作为企业级基础设施来处理流式数据非常有价值。

## 部署说明
* 该应用包含4个组件：kafka, kafka-dashboard, zookeeper, zookeeper-dashboard。
* 安装该应用后，先要将zookeeper的实例数改为3，且目前只支持3个实例，否则kafka可能启动失败。
* 在kafka中创建topic后，请小心使用伸缩功能，因为这可能影响您已保存到kafka中的数据，具体请参考[kafka分区与副本](https://kafka.apache.org/documentation)相关文档。

## 配置说明
配置项 | 默认值 | 描述
---|---|---
LISTENERS | PLAINTEXT://:9093 | broker服务监听端口
NUM_REPLICA_FETCHERS | 1 | 数据副本数量
LOG_RETENTION_HOURS | 168 | 日志文件删除之前保留的小时数
LOG_RETENTION_BYTES | -1 | 日志文件删除之前的最大大小，`-1`永不过期
DEBUG | true | 打印应用启动脚本的执行日志

如需设置更多参数，请参考[kafka官方文档](https://kafka.apache.org/documentation/#brokerconfigs)。

## 配置方式
1. 在[kafka官方文档](https://kafka.apache.org/documentation/#brokerconfigs)找到需要的配置项，假设为：`my.config.name`。
1. 将该配置项的名字全部大写且将`.`转为`_`，得到名字：`MY_CONFIG_NAME`。
1. 在云帮管理页面中找到应用，将该配置填入到应用的自定义环境变量中，如下图所示。
    ![app-add-env](http://grstatic.oss-cn-shanghai.aliyuncs.com/images/docs/common/app-add-env.jpg)
1. 重启应用生效。

## kafka-dashboard使用说明
1. 点击`kafka-dashboard`应用页面的访问按钮进入kafka仪表盘页面。
1. 点击`Add Cluster`按钮
1. 输入名字和zookeeper地址（固定地址：127.0.0.1:2181）
    ![add-cluster](http://grstatic.oss-cn-shanghai.aliyuncs.com/images/docs/common/kafka-dashboard-add-cluster.jpg)
1. 点击页面最下方的`Save`，然后就可以查看kafka集群的各个状态了。

## 基准测试

### 启动kafka
安装kafka，并将kafka实例数设置为3，每个实例分配1G内存。

### 启动kafka-client
因为kafka的生产者和消费者也会占用一部分内存，所以我们让这两个进程运行在一个单独的容器中，首先从源码安装kafka-clinet，并将内存设置为1G：
```
https://github.com/goodrain-apps/kafka.git?dir=client
```

以下测试命令全部在kafka-client容器中执行。

### 创建topic
```
kafka-topics.sh --create \
--zookeeper 127.0.0.1:2181 \
--replication-factor 3 \
--partitions 3 \
--topic test1
```

### 启动消费者
```
kafka-console-consumer.sh \
--zookeeper 127.0.0.1:2181 \
--from-beginning \
--topic test1 > /dev/null
```

### 启动生产者
在kafka容器中执行以下命令，获取kafka的所有节点：
```
net lookuprsv --format "%s:9093" $SERVICE_NAME
```

将得到的值替换到下列中的`<broker_list>`，然后在kafka-client中执行：
```
time kafka-producer-perf-test.sh \
--topic test1 \
--num-records $((10000)) \
--throughput $((100)) \
--record-size $((1024*100)) \
--producer-props \
acks=all \
bootstrap.servers=<broker_list> \
buffer.memory=$((1024*1024*1024)) \
compression.type=none \
batch.size=0
```

### 生产者执行结果
```
502 records sent, 100.3 records/sec (9.79 MB/sec), 16.6 ms avg latency, 179.0 max latency.
501 records sent, 100.1 records/sec (9.77 MB/sec), 4.5 ms avg latency, 47.0 max latency.
500 records sent, 99.9 records/sec (9.76 MB/sec), 3.6 ms avg latency, 34.0 max latency.
500 records sent, 99.9 records/sec (9.76 MB/sec), 57.1 ms avg latency, 696.0 max latency.
501 records sent, 100.0 records/sec (9.76 MB/sec), 4.0 ms avg latency, 39.0 max latency.
501 records sent, 100.0 records/sec (9.77 MB/sec), 3.0 ms avg latency, 24.0 max latency.
501 records sent, 100.1 records/sec (9.78 MB/sec), 3.0 ms avg latency, 22.0 max latency.
500 records sent, 99.9 records/sec (9.76 MB/sec), 4.0 ms avg latency, 70.0 max latency.
501 records sent, 100.1 records/sec (9.78 MB/sec), 3.0 ms avg latency, 12.0 max latency.
499 records sent, 99.6 records/sec (9.73 MB/sec), 3.5 ms avg latency, 47.0 max latency.
502 records sent, 100.3 records/sec (9.79 MB/sec), 4.0 ms avg latency, 56.0 max latency.
500 records sent, 100.0 records/sec (9.76 MB/sec), 3.8 ms avg latency, 58.0 max latency.
501 records sent, 100.1 records/sec (9.78 MB/sec), 3.3 ms avg latency, 34.0 max latency.
501 records sent, 100.1 records/sec (9.77 MB/sec), 3.4 ms avg latency, 29.0 max latency.
500 records sent, 99.9 records/sec (9.75 MB/sec), 3.3 ms avg latency, 36.0 max latency.
500 records sent, 100.0 records/sec (9.76 MB/sec), 19.2 ms avg latency, 527.0 max latency.
501 records sent, 100.1 records/sec (9.77 MB/sec), 3.2 ms avg latency, 30.0 max latency.
501 records sent, 100.0 records/sec (9.77 MB/sec), 61.4 ms avg latency, 677.0 max latency.
501 records sent, 100.0 records/sec (9.77 MB/sec), 4.3 ms avg latency, 61.0 max latency.
10000 records sent, 99.995000 records/sec (9.77 MB/sec), 10.61 ms avg latency, 696.00 ms max latency, 3 ms 50th, 14 ms 95th, 308 ms 99th, 649 ms 99.9th.

real    1m41.007s
user    0m9.354s
sys     0m2.324s
```

