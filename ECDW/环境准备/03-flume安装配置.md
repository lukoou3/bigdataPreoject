flume的安装比较简单，在flume-env.sh中配置jdk即可


## flume安装配置
### 解压
解压
```
[hadoop@hadoop101 software]$ ll
total 54408
-rw-rw-r-- 1 hadoop hadoop 55711670 Jun 30 07:05 apache-flume-1.7.0-bin.tar.gz
[hadoop@hadoop101 software]$ tar -zxf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
[hadoop@hadoop101 software]$ ll /opt/module
total 12
drwxrwxr-x  7 hadoop hadoop 4096 Jun 30 07:06 apache-flume-1.7.0-bin
drwxr-xr-x 11 hadoop hadoop 4096 Jun 30 04:18 hadoop-2.7.2
drwxr-xr-x  8 hadoop hadoop 4096 Jul 22  2017 jdk1.8.0_144
[hadoop@hadoop101 software]$ mv /opt/module/apache-flume-1.7.0-bin /opt/module/flume-1.7.0
[hadoop@hadoop101 software]$ ll /opt/module
total 12
drwxrwxr-x  7 hadoop hadoop 4096 Jun 30 07:06 flume-1.7.0
drwxr-xr-x 11 hadoop hadoop 4096 Jun 30 04:18 hadoop-2.7.2
drwxr-xr-x  8 hadoop hadoop 4096 Jul 22  2017 jdk1.8.0_144
```

### 配置
在flume-env.sh中配置JAVA_HOME
```
[hadoop@hadoop101 flume-1.7.0]$ cd conf/
[hadoop@hadoop101 conf]$ ll
total 16
-rw-r--r-- 1 hadoop hadoop 1661 Sep 26  2016 flume-conf.properties.template
-rw-r--r-- 1 hadoop hadoop 1455 Sep 26  2016 flume-env.ps1.template
-rw-r--r-- 1 hadoop hadoop 1565 Sep 26  2016 flume-env.sh.template
-rw-r--r-- 1 hadoop hadoop 3107 Sep 26  2016 log4j.properties
[hadoop@hadoop101 conf]$ cp flume-env.sh.template flume-env.sh
[hadoop@hadoop101 conf]$ vi flume-env.sh
#配置
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

修改日志的储存路径：
```
[hadoop@hadoop101 conf]$ vi log4j.properties
# 修改日志的储存路径，默认是./logs
flume.log.dir=/opt/module/flume-1.7.0/logs
```








