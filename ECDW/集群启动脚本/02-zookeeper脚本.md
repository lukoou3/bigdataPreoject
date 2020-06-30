## zookeeper启动停止脚本
```
[hadoop@hadoop101 app-script]$ vi zookeeper_cluster.sh
[hadoop@hadoop101 app-script]$ chmod +x zookeeper_cluster.sh 
[hadoop@hadoop101 app-script]$ ll
total 8
-rwxrwxr-x 1 hadoop hadoop 1394 Jul  1 06:41 hadoop_cluster.sh
-rwxrwxr-x 1 hadoop hadoop  781 Jul  1 06:54 zookeeper_cluster.sh
```

脚本：
```sh
#!/bin/bash

#集群启动异常解决，配置用户变量：1、cat /etc/profile >> ~/.bashrc    2、source ~/.bashrc
#环境变量配置到/etc/profile.d/env.sh就不用这么麻烦了
zookeeper_home=/opt/module/zookeeper-3.4.10
case $1 in
"start")
    for host in hadoop@hadoop101 hadoop@hadoop102 hadoop@hadoop103
    do
        ssh $host "$zookeeper_home/bin/zkServer.sh start"
    done
;;
"stop")
    for host in hadoop@hadoop101 hadoop@hadoop102 hadoop@hadoop103
    do
        ssh $host "$zookeeper_home/bin/zkServer.sh stop"
    done
;;
"status")
    for host in hadoop@hadoop101 hadoop@hadoop102 hadoop@hadoop103
    do
        ssh $host "$zookeeper_home/bin/zkServer.sh status"
    done
;;
*)
    echo "输入参数：start 或者 stop 或者 status"
    exit;
;;
esac
```

测试脚本：
```sh
[hadoop@hadoop101 app-script]$ ./zookeeper_cluster.sh start
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
6707 QuorumPeerMain
6759 Jps
--------- hadoop102 ----------
2578 QuorumPeerMain
2614 Jps
--------- hadoop103 ----------
2273 Jps
2243 QuorumPeerMain
[hadoop@hadoop101 app-script]$ ./zookeeper_cluster.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
[hadoop@hadoop101 app-script]$ ./zookeeper_cluster.sh stop
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
6894 Jps
--------- hadoop102 ----------
2724 Jps
--------- hadoop103 ----------
2383 Jps
```




