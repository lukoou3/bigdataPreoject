
## sqoop安装配置

### 下载并解压
1）下载地址：http://mirrors.hust.edu.cn/apache/sqoop/1.4.6/

2）上传安装包sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz到hadoop102的/opt/software路径中
```
[hadoop@hadoop101 software]$ ll
total 16480
drwxr-xr-x 3 hadoop hadoop     4096 Jul  5 07:08 mysql-libs
-rw-rw-r-- 1 hadoop hadoop 16870735 Jul  5 17:24 sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
```

3）解压sqoop安装包到指定目录，并修改目录名：
```
[hadoop@hadoop101 software]$ tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
[hadoop@hadoop101 software]$ cd /opt/module/
[hadoop@hadoop101 module]$ ll
total 36
drwxrwxr-x  8 hadoop hadoop 4096 Jul  3 07:02 flume-1.7.0
drwxr-xr-x 11 hadoop hadoop 4096 Jun 30 04:18 hadoop-2.7.2
drwxrwxr-x 11 hadoop hadoop 4096 Jul  5 07:55 hive-2.3.6
drwxr-xr-x  8 hadoop hadoop 4096 Jul 22  2017 jdk1.8.0_144
drwxr-xr-x  7 hadoop hadoop 4096 Jul  1 04:12 kafka_2.11-0.11.0.2
drwxr-xr-x  9 hadoop hadoop 4096 Apr 27  2015 sqoop-1.4.6.bin__hadoop-2.0.4-alpha
drwxr-xr-x  5 hadoop hadoop 4096 Dec 13  2017 tez-0.9.1
drwxrwxr-x  3 hadoop hadoop 4096 Jul  3 06:31 tomcat
drwxr-xr-x 12 hadoop hadoop 4096 Jun 30 07:56 zookeeper-3.4.10
[hadoop@hadoop101 module]$ mv sqoop-1.4.6.bin__hadoop-2.0.4-alpha/ sqoop-1.4.6
[hadoop@hadoop101 module]$ ls
flume-1.7.0  hadoop-2.7.2  hive-2.3.6  jdk1.8.0_144  kafka_2.11-0.11.0.2  sqoop-1.4.6  tez-0.9.1  tomcat  zookeeper-3.4.10
```

### 修改配置文件
重命名`sqoop-env.sh`配置文件
```
[hadoop@hadoop101 conf]$ mv sqoop-env-template.sh sqoop-env.sh
[hadoop@hadoop101 conf]$ vi sqoop-env.sh 
```

增加如下内容:
```sh
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
export HIVE_HOME=/opt/module/hive-2.3.6
```

### 拷贝JDBC驱动
拷贝jdbc驱动到sqoop的lib目录下:
```
[hadoop@hadoop101 conf]$ cp /opt/software/mysql-libs/mysql-connector-java-5.1.27-bin.jar  /opt/module/sqoop-1.4.6/lib
[hadoop@hadoop101 conf]$ ll /opt/module/sqoop-1.4.6/lib | grep mysql
-rw-r--r-- 1 hadoop hadoop  872303 Jul  5 17:36 mysql-connector-java-5.1.27-bin.jar
```

### 验证Sqoop
我们可以通过某一个command来验证sqoop配置是否正确：
```
[hadoop@hadoop101 sqoop-1.4.6]$ bin/sqoop help
```

出现一些Warning警告（警告信息已省略），并伴随着帮助命令的输出：
```
Available commands:
  codegen            Generate code to interact with database records
  create-hive-table  Import a table definition into Hive
  eval               Evaluate a SQL statement and display the results
  export             Export an HDFS directory to a database table
  help               List available commands
  import             Import a table from a database to HDFS
  import-all-tables  Import tables from a database to HDFS
  import-mainframe   Import datasets from a mainframe server to HDFS
  job                Work with saved jobs
  list-databases     List available databases on a server
  list-tables        List available tables in a database
  merge              Merge results of incremental imports
  metastore          Run a standalone Sqoop metastore
  version            Display version information

See 'sqoop help COMMAND' for information on a specific command.
```

### 测试Sqoop是否能够成功连接数据库
查询mysql中的数据库:
```
bin/sqoop list-databases --connect jdbc:mysql://hadoop101:3306/ --username root --password 123456
```

出现如下输出：
```
information_schema
hive
mysql
performance_schema
test
```

### 配置sqoop home
配置一下吧，方便一点：
```
[hadoop@hadoop101 sqoop-1.4.6]$ sudo vi /etc/profile.d/env.sh
```

添加：
```
#SQOOP_HOME
export SQOOP_HOME=/opt/module/sqoop-1.4.6
export PATH=$PATH:$SQOOP_HOME/bin
```







```

```







