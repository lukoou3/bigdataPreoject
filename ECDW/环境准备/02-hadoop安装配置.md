hadoop安装配置

## 解压

hadoop上传到opt目录下面的software文件夹下面，解压到/opt/module目录下
```
[hadoop@hadoop101 software]$ ll
total 193028
-rw-rw-r-- 1 hadoop hadoop 197657687 Jun 29 07:51 hadoop-2.7.2.tar.gz
[hadoop@hadoop101 software]$ tar -zxf hadoop-2.7.2.tar.gz -C /opt/module/
[hadoop@hadoop101 software]$ ll /opt/module/
```

配置环境变量
```
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

## 配置

### 核心配置文件
配置core-site.xml
```xml
<!-- 配置hdfs文件系统的命名空间-->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop101:9000</value>
</property>

<!-- 配置操作hdfs的缓冲大小-->
<property>
    <name>io.file.buffer.size</name>
    <value>4096</value>
</property>

<!-- 配置临时数据缓存目录-->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>

<property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
</property>
```

### HDFS配置文件
配置hadoop-env.sh
```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置hdfs-site.xml
```xml
<!-- hdfs的元数据存储位置 -->
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/opt/module/hadoop-2.7.2/data/dfs/name</value>
</property>

<!-- hdfs的数据存储位置 -->
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/opt/module/hadoop-2.7.2/data/dfs/data</value>
</property>

<!-- 副本数 -->
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>

<!-- 块大小 -->
<property>
    <name>dfs.blocksize</name>
    <value>134217728</value>
</property>

<!-- hdfs的namenode的web ui地址-->
<property>
    <name>dfs.http.address</name>
    <value>hadoop101:50070</value>
</property>

<!-- hdfs的secondarynamenode的web ui地址-->
<property>
    <name>dfs.secondary.http.address</name>
    <value>hadoop103:50090</value>
</property>

<!-- 是否开启web操作hdfs -->
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>

<!-- 是否启用hdfs的权限（acl） -->
<property> 
    <name>dfs.permissions.enabled</name> 
    <value>false</value> 
</property>
```

### YARN配置文件
配置yarn-env.sh
```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置yarn-site.xml
```xml
<!-- 指定YARN的老大（ResourceManager）的地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop102</value>
</property>

<!-- reducer获取数据的方式 -->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!-- 日志聚集功能使能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>

<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是 true。为了在虚拟机中测试。 -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是 true。为了在虚拟机中测试。 -->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```

### MapReduce配置文件
配置mapred-env.sh
```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置mapred-site.xml
```xml
<!-- 指定mapreduce运行框架  -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

<!-- 指定mapreduce历史服务的通信地址  -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop101:10020</value>
</property>

<!-- 指定mapreduce web ui地址  -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop101:19888</value>
</property>
```

### slaves配置文件
在该文件中增加如下内容：
```
hadoop101
hadoop102
hadoop103
```
**注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行**

## 启动集群

### 格式化NameNode
如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除data和log数据）.
```
[hadoop@hadoop101 hadoop-2.7.2]$ bin/hdfs namenode -format
```

### 启动HDFS
```
[hadoop@hadoop101 hadoop-2.7.2]$ sbin/start-dfs.sh
```

### 启动YARN
```
[hadoop@hadoop102 hadoop-2.7.2]$ sbin/start-yarn.sh
```
**注意：NameNode和ResourceManger如果不是同一台机器，不能在NameNode上启动YARN，应该在ResouceManager所在的机器上启动YARN。**


### 启动历史服务器
```
[hadoop@hadoop101 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
```



```

```




