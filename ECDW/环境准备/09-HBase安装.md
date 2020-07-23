
# HBase安装
Kylin依赖HBase，所以装一下HBase。

## 解压
```sh
[hadoop@hadoop101 software]$ ll
total 103244
-rw-rw-r-- 1 hadoop hadoop 105718722 Jul 24 07:16 hbase-1.3.1-bin.tar.gz
[hadoop@hadoop101 software]$ tar -zxf hbase-1.3.1-bin.tar.gz -C /opt/module/
[hadoop@hadoop101 software]$ ll /opt/module/
drwxr-xr-x 11 hadoop hadoop 4096 Jun 30 04:18 hadoop-2.7.2
drwxrwxr-x  7 hadoop hadoop 4096 Jul 24 07:16 hbase-1.3.1
```

## 配置

### hbase-env.sh
```
export JAVA_HOME=/opt/module/jdk1.8.0_144
# 禁用自带的zk
export HBASE_MANAGES_ZK=false

#JDK1.8需要注释
#export HBASE_MASTER_OPTS。。。。
#export HBASE_REGIONSERVER_OPTS。。。
```

### hbase-site.xml
```xml
<configuration>
	<property>     
		<name>hbase.rootdir</name>     
		<value>hdfs://hadoop101:9000/hbase</value>   
	</property>

	<property>   
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>

   <!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
	<property>
		<name>hbase.master.port</name>
		<value>16000</value>
	</property>

	<property>   
		<name>hbase.zookeeper.quorum</name>
	     <value>hadoop101,hadoop102,hadoop103</value>
	</property>

	<property>   
		<name>hbase.zookeeper.property.dataDir</name>
	     <value>/opt/module/zookeeper-3.4.10/data</value>
	</property>
</configuration>
```
### regionservers
```
hadoop101
hadoop102
hadoop103
```

### 软连接hadoop配置文件到hbase
```
[hadoop@hadoop101 conf]$ ln -s /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml /opt/module/hbase-1.3.1/conf/core-site.xml
[hadoop@hadoop101 conf]$ ll
total 40
lrwxrwxrwx 1 hadoop hadoop   49 Jul 24 07:33 core-site.xml -> /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml
-rw-r--r-- 1 hadoop hadoop 1811 Sep 21  2016 hadoop-metrics2-hbase.properties
-rw-r--r-- 1 hadoop hadoop 4537 Nov  7  2016 hbase-env.cmd
-rw-r--r-- 1 hadoop hadoop 7503 Jul 24 07:25 hbase-env.sh
-rw-r--r-- 1 hadoop hadoop 2257 Sep 21  2016 hbase-policy.xml
-rw-r--r-- 1 hadoop hadoop 1551 Jul 24 07:29 hbase-site.xml
-rw-r--r-- 1 hadoop hadoop 4722 Apr  5  2017 log4j.properties
-rw-r--r-- 1 hadoop hadoop   30 Jul 24 07:30 regionservers
[hadoop@hadoop101 conf]$ 
```

## 同步HBase目录到其他两个节点
```
[hadoop@hadoop101 module]$ xsync hbase-1.3.1/
```

## 启动

###  Zookeeper正常部署
首先保证Zookeeper集群的正常部署，并启动之：
```
[hadoop@hadoop101 app-script]$ ./zookeeper_cluster.sh start
```

### Hadoop正常部署
Hadoop集群的正常部署并启动：
```
[hadoop@hadoop101 app-script]$ ./hadoop_cluster.sh start
```

### 启动方式1
```
[hadoop@hadoop101 hbase-1.3.1]$ bin/hbase-daemon.sh start master
[hadoop@hadoop101 hbase-1.3.1]$ bin/hbase-daemon.sh start regionserver
```

提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。

修复提示：

a、同步时间服务

b、属性：hbase.master.maxclockskew设置更大的值
```
<property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
 </property>
```

### 启动方式2
```
[hadoop@hadoop101 hbase-1.3.1]$ bin/start-hbase.sh
```
对应的停止服务：
```
[hadoop@hadoop101 hbase-1.3.1]$ bin/stop-hbase.sh
```

### 检查各进程是否启动正常
主节点和备用节点都启动 hmaster 进程

各从节点都启动 hregionserver 进程
```
[hadoop@hadoop101 hbase-1.3.1]$ xcall jps
--------- hadoop101 ----------
1344 QuorumPeerMain
2400 HRegionServer
1969 JobHistoryServer
1606 DataNode
1894 NodeManager
2265 HMaster
1497 NameNode
2794 Jps
--------- hadoop102 ----------
1216 DataNode
1345 ResourceManager
2022 Jps
1159 QuorumPeerMain
1803 HRegionServer
1453 NodeManager
--------- hadoop103 ----------
1217 DataNode
1154 QuorumPeerMain
1767 Jps
1402 NodeManager
1324 SecondaryNameNode
1566 HRegionServer
```

### 查看HBase页面
启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：
http://hadoop101:16010 

## 环境变量配置
```
[hadoop@hadoop101 module]$ sudo vi /etc/profile.d/env.sh

#HBASE_HOME
export HBASE_HOME=/opt/module/hbase-1.3.1
export PATH=$PATH:$HBASE_HOME/bin
```


```

```