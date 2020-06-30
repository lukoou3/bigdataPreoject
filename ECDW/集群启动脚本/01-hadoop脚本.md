## hadoop启动停止脚本
创建app-script目录用于存放集群启动脚本。
```
[hadoop@hadoop101 ~]$ mkdir app-script
[hadoop@hadoop101 ~]$ cd app-script/
[hadoop@hadoop101 app-script]$ vi hadoop_cluster.sh
[hadoop@hadoop101 app-script]$ chmod +x hadoop_cluster.sh
```

脚本：
```sh
#!/bin/bash
hadoop_home=/opt/module/hadoop-2.7.2
function start_hadoop_cluster()
{
    echo "start_hadoop_cluster ................"
    
    echo "=====================    正在启动hdfs    ====================="
    ssh hadoop@hadoop101 "$hadoop_home/sbin/start-dfs.sh"

    echo "=====================    正在启动yarn    ====================="
    ssh hadoop@hadoop102 "$hadoop_home/sbin/start-yarn.sh"

    echo "================    正在启动historyserver    =================="
    ssh hadoop@hadoop101 "$hadoop_home/sbin/mr-jobhistory-daemon.sh start historyserver"
    
    echo "start_hadoop_cluster finished................"
}

function stop_hadoop_cluster()
{
    echo "stop_hadoop_cluster ................"
    
    echo "=====================    正在关闭yarn    ====================="
    ssh hadoop@hadoop102 "$hadoop_home/sbin/stop-yarn.sh"
    
    echo "================    正在关闭historyserver    =================="
    ssh hadoop@hadoop101 "$hadoop_home/sbin/mr-jobhistory-daemon.sh stop historyserver"  
    
    echo "=====================    正在关闭hdfs    ====================="
    ssh hadoop@hadoop101 "$hadoop_home/sbin/stop-dfs.sh"
    
    echo "stop_hadoop_cluster finished................"
}

case $1 in
"start")
    start_hadoop_cluster
;;
"stop")
    stop_hadoop_cluster
;;
*)
    echo "输入参数：start 或者 stop"
    exit;
;;
esac

```

脚本测试：
```sh
[hadoop@hadoop101 app-script]$ ./hadoop_cluster.sh start
start_hadoop_cluster ................
=====================    正在启动hdfs    =====================
Starting namenodes on [hadoop101]
hadoop101: starting namenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-hadoop-namenode-hadoop101.out
hadoop101: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-hadoop-datanode-hadoop101.out
hadoop102: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-hadoop-datanode-hadoop102.out
hadoop103: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-hadoop-datanode-hadoop103.out
Starting secondary namenodes [hadoop103]
hadoop103: starting secondarynamenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-hadoop-secondarynamenode-hadoop103.out
=====================    正在启动yarn    =====================
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-hadoop-resourcemanager-hadoop102.out
hadoop102: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-hadoop-nodemanager-hadoop102.out
hadoop101: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-hadoop-nodemanager-hadoop101.out
hadoop103: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-hadoop-nodemanager-hadoop103.out
================    正在启动historyserver    ==================
starting historyserver, logging to /opt/module/hadoop-2.7.2/logs/mapred-hadoop-historyserver-hadoop101.out
start_hadoop_cluster finished................
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
5745 DataNode
6005 NodeManager
6248 Jps
5610 NameNode
6076 JobHistoryServer
--------- hadoop102 ----------
2048 NodeManager
1938 ResourceManager
1810 DataNode
2362 Jps
--------- hadoop103 ----------
1973 NodeManager
2102 Jps
1853 SecondaryNameNode
1790 DataNode
[hadoop@hadoop101 app-script]$ ./hadoop_cluster.sh stop
stop_hadoop_cluster ................
=====================    正在关闭yarn    =====================
stopping yarn daemons
stopping resourcemanager
hadoop102: stopping nodemanager
hadoop101: stopping nodemanager
hadoop103: stopping nodemanager
no proxyserver to stop
================    正在关闭historyserver    ==================
stopping historyserver
=====================    正在关闭hdfs    =====================
Stopping namenodes on [hadoop101]
hadoop101: stopping namenode
hadoop101: stopping datanode
hadoop102: stopping datanode
hadoop103: stopping datanode
Stopping secondary namenodes [hadoop103]
hadoop103: stopping secondarynamenode
stop_hadoop_cluster finished................
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
6664 Jps
--------- hadoop102 ----------
2546 Jps
--------- hadoop103 ----------
2211 Jps
[hadoop@hadoop101 app-script]$ 
```

