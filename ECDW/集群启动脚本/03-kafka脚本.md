## kafka启动停止脚本
```sh
[hadoop@hadoop101 app-script]$ vi kafka_cluster.sh
[hadoop@hadoop101 app-script]$ chmod +x kafka_cluster.sh 
[hadoop@hadoop101 app-script]$ ll
total 12
-rwxrwxr-x 1 hadoop hadoop 1394 Jul  1 06:41 hadoop_cluster.sh
-rwxrwxr-x 1 hadoop hadoop  480 Jul  1 07:09 kafka_cluster.sh
-rwxrwxr-x 1 hadoop hadoop  781 Jul  1 06:54 zookeeper_cluster.sh
```

脚本：
```sh
#!/bin/bash
kafka_home=/opt/module/kafka_2.11-0.11.0.2
case $1 in
"start")
    for host in hadoop@hadoop101 hadoop@hadoop102 hadoop@hadoop103
    do
        ssh $host "$kafka_home/bin/kafka-server-start.sh -daemon $kafka_home/config/server.properties"
    done
;;
"stop")
    for host in hadoop@hadoop101 hadoop@hadoop102 hadoop@hadoop103
    do
        ssh $host "$kafka_home/bin/kafka-server-stop.sh"
    done
;;
*)
    echo "输入参数：start 或者 stop"
    exit;
;;
esac
```

测试脚本：
```
[hadoop@hadoop101 app-script]$ ./kafka_cluster.sh start
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
7633 Jps
7555 Kafka
7244 QuorumPeerMain
--------- hadoop102 ----------
3347 Kafka
3044 QuorumPeerMain
3412 Jps
--------- hadoop103 ----------
3000 Kafka
3065 Jps
2703 QuorumPeerMain
[hadoop@hadoop101 app-script]$ ./kafka_cluster.sh stop
[hadoop@hadoop101 app-script]$ xcall jps
--------- hadoop101 ----------
7244 QuorumPeerMain
7806 Jps
--------- hadoop102 ----------
3044 QuorumPeerMain
3500 Jps
--------- hadoop103 ----------
3152 Jps
2703 QuorumPeerMain
```

