zookeeper的安装比较简单，**需要注意的一点是myid的配置不能重复**。

## 集群规划
在hadoop101、hadoop102和hadoop103三个节点上部署Zookeeper。


## zookeeper安装配置

### 解压
```
[hadoop@hadoop101 software]$ tar -zxf zookeeper-3.4.10.tar.gz -C /opt/module/
[hadoop@hadoop101 software]$ ll /opt/module/
total 16
drwxrwxr-x  7 hadoop hadoop 4096 Jun 30 07:06 flume-1.7.0
drwxr-xr-x 11 hadoop hadoop 4096 Jun 30 04:18 hadoop-2.7.2
drwxr-xr-x  8 hadoop hadoop 4096 Jul 22  2017 jdk1.8.0_144
drwxr-xr-x 10 hadoop hadoop 4096 Mar 23  2017 zookeeper-3.4.10
```

### 配置
#### 配置服务器编号myid
创建data目录，在其下创建myid文件，文件中写入编号1。**myid一定要在linux里面创建，在notepad++里面很可能乱码**。
```
[hadoop@hadoop101 zookeeper-3.4.10]$ mkdir data
[hadoop@hadoop101 zookeeper-3.4.10]$ vi data/myid
[hadoop@hadoop101 zookeeper-3.4.10]$ cat data/myid 
1
```

#### 配置zoo.cfg文件
```
[hadoop@hadoop101 zookeeper-3.4.10]$ cd conf
[hadoop@hadoop101 conf]$ ll
total 12
-rw-rw-r-- 1 hadoop hadoop  535 Mar 23  2017 configuration.xsl
-rw-rw-r-- 1 hadoop hadoop 2161 Mar 23  2017 log4j.properties
-rw-rw-r-- 1 hadoop hadoop  922 Mar 23  2017 zoo_sample.cfg
[hadoop@hadoop101 conf]$ cp zoo_sample.cfg zoo.cfg
[hadoop@hadoop101 conf]$ vi zoo.cfg 
```
添加的配置：
```
clientPort=2181
dataDir=/opt/module/zookeeper-3.4.10/data
dataLogDir=/opt/module/zookeeper-3.4.10/log
server.1=hadoop101:2888:3888
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
```

配置参数解读
```
server.A=B:C:D

A是一个数字，表示这个是第几号服务器；
集群模式下配置一个文件myid，这个文件在data目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
B是这个服务器的ip地址；
C是这个服务器与集群中的Leader服务器交换信息的端口；
D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
```

#### 同步zookeeper目录到其他两个节点
```
[hadoop@hadoop101 module]$ xsync zookeeper-3.4.10/
```

#### 修改各个myid
hadoop101为1，hadoop102为2，hadoop103为3。

#### 环境变量配置
```
[hadoop@hadoop101 module]$ sudo vi /etc/profile.d/env.sh

#ZOOKEEPER_HOME
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

## 集群启动测试
需要注意的是会在命令的路径生成zookeeper.out文件，要保证有可以在相应的目录有创建文件的权限。

```
# 启动
xcall /opt/module/zookeeper-3.4.10/bin/zkServer.sh start
# 查看状态
xcall /opt/module/zookeeper-3.4.10/bin/zkServer.sh status
# 停止
xcall /opt/module/zookeeper-3.4.10/bin/zkServer.sh stop
```

修改zookeeper-3.4.10/conf/log4j.properties，还是没用，先不关了。
```
zookeeper.log.dir=/opt/module/zookeeper-3.4.10/logs
```




```

```