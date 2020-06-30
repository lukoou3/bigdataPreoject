kafka安装配置主要需要注意**`broker.id`不能重复**。

## 集群规划
在hadoop101、hadoop102和hadoop103三个节点上部署kafka。

## kafka安装配置

### 解压
```
[hadoop@hadoop101 software]$ tar -zxf kafka_2.11-0.11.0.2.tgz -C /opt/module/
[hadoop@hadoop101 software]$ ll /opt/module/
total 60
drwxrwxr-x  7 hadoop hadoop  4096 Jun 30 07:06 flume-1.7.0
drwxr-xr-x 11 hadoop hadoop  4096 Jun 30 04:18 hadoop-2.7.2
drwxr-xr-x  8 hadoop hadoop  4096 Jul 22  2017 jdk1.8.0_144
drwxr-xr-x  6 hadoop hadoop  4096 Nov 11  2017 kafka_2.11-0.11.0.2
drwxr-xr-x 12 hadoop hadoop  4096 Jun 30 07:56 zookeeper-3.4.10
-rw-rw-r--  1 hadoop hadoop 40480 Jun 30 08:18 zookeeper.out
```

### 配置

#### 创建logs文件夹
这里用于存放kafka中的topic。
```
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ mkdir logs
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ ls
bin  config  libs  LICENSE  logs  NOTICE  site-docs
```

#### 修改server.properties配置文件
```shell
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ cd config/
[hadoop@hadoop101 config]$ vi server.properties 
```

输入以下内容：
```python
#broker的全局唯一编号，不能重复
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径	
log.dirs=/opt/module/kafka_2.11-0.11.0.2/logs
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop101:2181,hadoop102:2181,hadoop103:2181
```

#### 环境变量配置
```
[hadoop@hadoop101 module]$ sudo vi /etc/profile.d/env.sh

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka_2.11-0.11.0.2
export PATH=$PATH:$KAFKA_HOME/bin
```

#### 同步zookeeper目录到其他两个节点
```
[hadoop@hadoop101 module]$ xsync kafka_2.11-0.11.0.2/
```

#### 修改各个broker.id
hadoop101为broker.id=0，hadoop102为broker.id=1，hadoop103为broker.id=2。


## 集群启动测试
### 先启动zkServer
需要先启动zkServer，zkServer命令：
```
./zkServer.sh start
./zkServer.sh stop
./zkServer.sh restart
./zkServer.sh status
```

依次在三台电脑上启动zookeper：
zkServer.sh start
查看启动状态：
zkServer.sh status

进程查看带全类名：
jps -l

```
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ xcall zkServer.sh start
--------- hadoop101 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
--------- hadoop102 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
--------- hadoop103 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ xcall zkServer.sh status
--------- hadoop101 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
--------- hadoop102 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
--------- hadoop103 ----------
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader
```

### 后台启动kafka
```
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ bin/kafka-server-start.sh -daemon config/server.properties
[hadoop@hadoop102 kafka_2.11-0.11.0.2]$ bin/kafka-server-start.sh -daemon config/server.properties
[hadoop@hadoop103 kafka_2.11-0.11.0.2]$ bin/kafka-server-start.sh -daemon config/server.properties
```
-daemon 表示守护进程，会将日志打印在后台。

### 关闭Kafka集群
```
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ bin/kafka-server-stop.sh stop
[hadoop@hadoop102 kafka_2.11-0.11.0.2]$ bin/kafka-server-stop.sh stop
[hadoop@hadoop103 kafka_2.11-0.11.0.2]$ bin/kafka-server-stop.sh stop
```

## kafka命令测试
### 创建topic
```
bin/kafka-topics.sh --zookeeper hadoop101:2181,hadoop102:2181 --create  --replication-factor 2 --partitions 3 --topic test-topic
```

### 查看所有topic
```
bin/kafka-topics.sh --zookeeper hadoop101:2181 --list

bin/kafka-topics.sh --zookeeper hadoop101:2181,hadoop102:2181 --list
```

### 查看topic详情
```
[hadoop@hadoop101 kafka_2.11-0.11.0.2]$ bin/kafka-topics.sh --zookeeper hadoop101:2181,hadoop102:2181 --describe --topic test-topic
Topic:test-topic	PartitionCount:3	ReplicationFactor:2	Configs:
	Topic: test-topic	Partition: 0	Leader: 2	Replicas: 2,0	Isr: 2,0
	Topic: test-topic	Partition: 1	Leader: 0	Replicas: 0,1	Isr: 0,1
	Topic: test-topic	Partition: 2	Leader: 1	Replicas: 1,2	Isr: 1,2
```

### 生产消费者测试
消费：
```
bin/kafka-console-consumer.sh --zookeeper hadoop101:2181 --from-beginning --topic test-topic
bin/kafka-console-consumer.sh --bootstrap-server hadoop101:9092 --from-beginning --topic test-topic
```

生产：
```
bin/kafka-console-producer.sh --broker-list hadoop101:9092 --topic test-topic
```

### 删除topic
必须配置`delete.topic.enable=true`才能删除topic(删除会有几分钟的延迟)，不然这是标记。
```
bin/kafka-topics.sh --zookeeper hadoop101:2181,hadoop102:2181 --delete --topic test-topic
```

## Kafka压力测试
用Kafka官方自带的脚本，对Kafka进行压测。Kafka压测时，可以查看到哪个地方出现了瓶颈（`CPU，内存，网络IO`）。**一般都是网络IO达到瓶颈**。

```
kafka-consumer-perf-test.sh
kafka-producer-perf-test.sh
```

### Kafka Producer压力测试
（1）在/opt/module/kafka/bin目录下面有这两个文件。我们来测试一下
```
bin/kafka-producer-perf-test.sh  --topic test --record-size 100 --num-records 100000 --throughput -1 --producer-props bootstrap.servers=hadoop101:9092,hadoop102:9092,hadoop103:9092
```
说明：
record-size是一条信息有多大，单位是字节。
num-records是总共发送多少条信息。
throughput 是每秒多少条信息，设成-1，表示不限流，可测出生产者最大吞吐量。

2）Kafka会打印下面的信息
```
100000 records sent, 95877.277085 records/sec (9.14 MB/sec), 187.68 ms avg latency, 424.00 ms max latency, 155 ms 50th, 411 ms 95th, 423 ms 99th, 424 ms 99.9th.
```
参数解析：本例中一共写入10w条消息，吞吐量为9.14 MB/sec，每次写入的平均延迟为187.68毫秒，最大的延迟为424.00毫秒。


### Kafka Consumer压力测试
Consumer的测试，如果这四个指标（IO，CPU，内存，网络）都不能改变，考虑增加分区数来提升性能。
```
bin/kafka-consumer-perf-test.sh --zookeeper hadoop101:2181 --topic test --fetch-size 10000 --messages 10000000 --threads 1
```
参数说明：
--zookeeper 指定zookeeper的链接信息
--topic 指定topic的名称
--fetch-size 指定每次fetch的数据的大小
--messages 总共要消费的消息个数

测试结果说明：
```
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
2019-02-19 20:29:07:566, 2019-02-19 20:29:12:170, 9.5368, 2.0714, 100010, 21722.4153
```
开始测试时间，测试结束数据，共消费数据9.5368MB，吞吐量2.0714MB/s，共消费100010条，平均每秒消费21722.4153条。




```

```








