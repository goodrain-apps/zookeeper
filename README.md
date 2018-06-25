# rainbond-zookeeper

## 什么是zookeeper
Apache ZooKeeper是Apache软件基金会的一个软件项目，他为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。ZooKeeper曾经是Hadoop的一个子项目，但现在是一个独立的顶级项目。

ZooKeeper的架构通过冗余服务实现高可用性。因此，如果第一次无应答，客户端就可以询问另一台ZooKeeper主机。ZooKeeper节点将它们的数据存储于一个分层的命名空间，非常类似于一个文件系统或一个前缀树结构。客户端可以在节点读写，从而以这种方式拥有一个共享的配置服务。更新是全序的。

## 部署说明
安装该应用后，请将`zookeeper`的实例数改为3，否则`zookeeper`无法正常启动。

## 修改实例数说明
因为zookeeper应用具有特殊性，目前还不直接支持zookeeper的伸缩操作，如果要扩大或缩小zookeeper集群的规模，请按以下方式进行：
1. 进入zookeeper应用管理页面，点击关闭按钮将zookeeper停止。
1. 点击设置，修改环境变量ZK_REPLICAS的值为目标实例数。
1. 点击伸缩，修改实例数量的值为目标实例数，要与上一步中的值相同。
1. 最后点击启动，等待应用启动完毕。

## 配置说明

配置项 | 默认值 | 描述
---|---|---
ZK_CLIENT_PORT | 2181 | 客户端连接端口
ZK_SERVER_PORT | 2888 | 节点间通信端口
ZK_ELECTION_PORT | 3888 | 选举端口
ZK_MIN_SESSION_TIMEOUT | 4000 | 最小会话超时
ZK_MAX_SESSION_TIMEOUT | 40000 | 最大会话超时
ZK_LOG_LEVEL | INFO | 日志输出级别

如需设置更多参数，请参考[zookeeper官方文档](https://zookeeper.apache.org/doc/r3.4.12/zookeeperAdmin.html#sc_configuration)。

## 配置方式
1. 在[zookeeper官方文档](https://zookeeper.apache.org/doc/r3.4.12/zookeeperAdmin.html#sc_configuration)找到需要的配置项，假设为：`my.config.name`。
1. 将该配置项的名字全部大写且将`.`转为`_`，得到名字：`MY_CONFIG_NAME`。
1. 在云帮管理页面中找到应用，将该配置填入到应用的自定义环境变量中，如下图所示。
    ![app-add-env](http://grstatic.oss-cn-shanghai.aliyuncs.com/images/docs/common/app-add-env.jpg)
1. 重启应用生效。

## 基准测试

### 开始测试
进入任意zookeeper节点执行以下命令：
```
cat > test.sh <<'EOF'
#!/bin/bash
zkCli.sh rmr /test &> /dev/null
zkCli.sh create /test 0 &> /dev/null
for i in `seq 1 100`; do
  ok=`zkCli.sh create /test/$i $i 2>&1 | tail -1 | grep -e "Created " | wc -l`
  if [[ x$ok == x1 ]]; then
      echo -e "$i\tcreate\tsuccess"
  else
      echo -e "$i\tcreate\tfailed"
  fi
  
  ok=`zkCli.sh rmr /test/$i 2>&1 | tail -1 | grep -e "WatchedEvent " | wc -l`
  if [[ x$ok == x1 ]]; then
      echo -e "$i\tdelete\tsuccess"
  else
      echo -e "$i\tdelete\tfailed"
  fi
done

zkCli.sh rmr /test &> /dev/null
EOF
chmod +x test.sh
{ time ./test.sh; } &> /var/lib/zookeeper/result.log &
```

### 执行结果
```
tail /var/lib/zookeeper/result.log
98      create  success
98      delete  success
99      create  success
99      delete  success
100     create  success
100     delete  success

real    2m21.844s
user    1m21.866s
sys     0m23.583s
```

