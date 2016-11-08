---
layout: post
title: Storm 集群部署
date: 2016-09-27
categories: [大数据技术,storm]
tags: [部署,storm]
description: 

---

---------

## 一、 storm简介

Storm是一个免费开源、分布式、高容错的实时计算系统。Storm令持续不断的流计算变得容易，弥补了Hadoop批处理所不能满足的实时要求。Storm经常用于在实时分析、在线机器学习、持续计算、分布式远程调用和ETL等领域。Storm的部署管理非常简单，而且，在同类的流式计算工具，Storm的性能也是非常出众的。实现一个实时计算系统要解决的问题有：低延迟、高性能、分布式、可扩展、容错性。

### storm优势

 - 简单的编程模型,MR降低了并行批处理的复杂性,Storm降低了进行实时处理的复杂性
 - 服务化：一个服务框架，支持热部署时，即时上线或下线APP
 - 支持各种编程语言，默认支持Java，ruby，Python，clojure。增加其他语言支持，只需实现一个简单的storm通信协议即可。
 - 容错性：storm会管理工作进程和节点的故障
 - 水平扩展：计算是在多个线程、进程和服务器之间并行进行的。
 - 可靠的消息处理：storm保证每个消息至少能得到一次完整处理,任务失败会负责从消息源重试消息
 - 快速：底层消息队列是ZeroMQ
 - 本地模式 ：模拟Storm集群

### Storm架构

集群由一个主节点和多个工作节点组成（master和slaver的架构）。主节点运行了一个Nimbus的守护进程，用于分配代码、布置任务及故障检测。每个工作节点都运行了一个名为Supervisor的守护进程，用于监听工作，开始并终止工作进程。Nimbus和supervisor都是无状态的，都能快速失败，十分健壮。两者的协调工作是由Zookeeper来完成的。Zookeeper用于管理集群中的不同组件，ZeroMQ是内部消息系统，JZMQ是ZeroMQ的Java binding。AWS上一键部署Storm集群用的是storm-deploy的子项目。

Hadoop和Storm的几个名词的对比
```
Jobtracker		nimbus
Tasktracker		supervisor
Child			worker（是一个进程）
Job				topology
Mapper/reducer	spout/bolt
Task            每一个spout/bolt的线程称为一个task, 多个task可能共享一个物理线程，称为一个executor
```

- Topology:storm中运行的一个实时应用程序，各个组件间的消息流动形成逻辑上的一个拓扑结构
- Spout：从外部数据源中读取数据转化为拓扑的源数据，有个nextTuple（）方法，死循环，不停的调用
- Bolt:接收源数据然后执行逻辑运算，可以执行过滤，函数操作，合并等。有个execute方法（Tuple input），接收数据后调用这个方法，在其中写自己的逻辑
Tuple:一次消息传递的基本单元，本来应该是一个key-value的map，但是由于tuple的字段需要事先定义好，所以tuple是一个value list。

## 二、 集群部署

### 依赖
- JDK：`1.7.0`
- Python：`2.7.5`
- Zeromq: `2.1.7`
- Jzmq: `master`
- Zookeeper: `3.4.5`

### 集群环境

* 操作系统：centos7
* 账户：root
* 主机
```
kafka04 master
kafka05 worker
kafka06 worker
```
> 机器: kafka04、kafka05、kafka06共计三台机器。其中kafka04为主节点，kafka05-kafka06为工作节点。kafka04连接外网，kafka05-kafka06通过千兆交换机与kafka04相连。防火墙关闭


### 部署过程

- **1. JDK安装**

- **2. Python安装**
 
- **3. Zeromq的安装**

下载Zeromq源码包，解压并安装（master和worker都要安装）
```
tar -zxvf zeromq-2.1.7.tar.gz
cd ./zeromq-2.1.7
./configure
make
make install
```

- **4.下载jzmq安装包，解压并安装（master和worker都要安装）**
```
unzip  jzmq-master.zip
cd jzmq-jni/
./autogen.sh
./configure
make
make install
```

 **注意** : Make的时候如果报错**make[1]: No rule to make target \`classdist_noinst.stamp', needed by \`org/zeromq/ZMQ.class'. Stop.**  解决方法：
```
touch src/classdist_noinst.stamp 
cd src/org/zeromq/ 
$ /jzmq/src/org/zeromq
javac *.java
```

- **5. zookeeper的部署**

- **6. 下载storm，unzip解压**。（master和worker都要部署）
修改配置文件./conf/storm.yaml（注意每一行最前面不能有空格，要删掉最前面的空格。否则报错）
```
## These MUST be filled in for a storm configuration
storm.zookeeper.servers:
    - "kafka04"
    - "kafka05"
    - "kafka06"
# 
nimbus.seeds: ["kafka04"]
storm.local.dir:"/usr/local/storm" 
ui.port: "8080"
supervisor.slots.ports: 
    - 6700
    - 6701
    - 6702
    - 6703
```

**注**：根据你的cpu的能力可以多几个端口，每一个端口对应storm一个slot，可以运行storm的一个bolt，此处开设了四个端口。

> zookeeper.server就是之前配置的。Nimbus.host是选取的master，storm.local.dir存储了一些storm的信息，可以随意设置，此处设置为/usr/local/storm。Ui.port指的是运行拓扑的时候的web观察端口，可在浏览器里看到。没有拓扑时通过master的地址:8080端口查看集群。

将storm下的bin文件夹加入/etc/profile文件的path中，并source一下使之生效
`source /etc/profile`

- **7. 集群启动**

在 master上执行
```
nohup storm nimbus &   //开启master的主守护进程
nohup storm ui  &
```
Jps查看出现nimbus，core

ui可以通过Kafka04:8080查看

在worker节点上执行
```
nohup storm supervisor & //开启工作节点的守护进程
```
Jps查看出现 supervisor

日志文件可以从`$STORM_HOME/logs`下查看

- **8. 测试**
在Storm的安装目录中自带了一系列的Storm例子代码和打包好的jar包，其中包括最著名的WordCount实例。

**启动WordCount**

```
bin/storm jar examples/storm-starter/storm-starter-topologies-1.0.2.jar org.apache.storm.starter.WordCountTopology WordCountTopology
```
logs目录下会产生对应worker的日志。`tail -f `查看
```
tail -f ./logs/workers-artifacts/WordCountTopology-1-1474988850/6700/worker.log
```

**查看Storm任务运行情况**

查看当前正在运行的Topology: `bin/storm list`
```
Topology_name        Status     Num_tasks  Num_workers  Uptime_secs
-------------------------------------------------------------------
WordCountTopology    ACTIVE     0          0            8
```
杀死正在执行的Topology: `bin/storm kill WordCountTopology`

**Storm UI**
使用./storm ui启动UI进行后，可以访问http://kafka04:8080来查看Storm的运行状态





