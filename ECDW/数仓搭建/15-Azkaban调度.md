# Azkaban调度

## 创建MySQL数据库和表
创建gmall_report数据库：
![](assets/markdown-img-paste-2020072022360795.png)

由于时间关系，就建几个表，用于导出，都是一样的。数据表的字段要求和hdfs中的每行的数据字段一样。


```sql
CREATE TABLE `ads_uv_count` (
  `dt` date NOT NULL,
  `day_count` bigint(20) DEFAULT NULL,
  `wk_count` bigint(20) DEFAULT NULL,
  `mn_count` bigint(20) DEFAULT NULL,
  `is_weekend` varchar(2) DEFAULT NULL,
  `is_monthend` varchar(2) DEFAULT NULL,
  PRIMARY KEY (`dt`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `ads_user_retention_day_rate` (
  `stat_date` date NOT NULL,
  `create_date` date NOT NULL,
  `retention_day` int(11) NOT NULL,
  `retention_count` bigint(20) DEFAULT NULL,
  `new_mid_count` bigint(20) DEFAULT NULL,
  `retention_ratio` double DEFAULT NULL,
  PRIMARY KEY (`create_date`,`retention_day`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `ads_user_topic` (
  `dt` date NOT NULL,
  `day_users` bigint(20) DEFAULT NULL,
  `day_new_users` bigint(20) DEFAULT NULL,
  `day_new_payment_users` bigint(20) DEFAULT NULL,
  `payment_users` bigint(20) DEFAULT NULL,
  `users` bigint(20) DEFAULT NULL,
  `day_users2users` double DEFAULT NULL,
  `payment_users2users` double DEFAULT NULL,
  `day_new_users2users` double DEFAULT NULL,
  PRIMARY KEY (`dt`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## Sqoop导出脚本
### 脚本
在`/home/hadoop/bigdata-project/ecdw/sqoop`下创建sqoop_export.sh文件
```sh
[hadoop@hadoop101 sqoop]$ pwd
/home/hadoop/bigdata-project/ecdw/sqoop
[hadoop@hadoop101 sqoop]$ vi sqoop_export.sh
```

内容：
```sh
#!/bin/bash

sqoop_home=/opt/module/sqoop-1.4.6

#不同于导入，导出的都是hive表中的全量数据，不像导入的时候可以自定义sql查询


# 导出函数。$1：hive中ads表的名称，$2：mysql中表的名称，$3：mysql表中决定插入还是更新的属性列。
export_data() {
$sqoop_home/bin/sqoop export \
--connect "jdbc:mysql://hadoop101:3306/gmall_report?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password 123456 \
--table $2 \
--num-mappers 1 \
--export-dir /warehouse/gmall/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $3 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}

case $1 in
"ads_uv_count")
     export_data "ads_uv_count" "ads_uv_count" "dt"
;;
"ads_user_retention_day_rate") 
     export_data "ads_user_retention_day_rate" "ads_user_retention_day_rate" "create_date,retention_day"
;;
"ads_user_topic")
     export_data "ads_user_topic" "ads_user_topic" "dt"
;;
"all")
     export_data "ads_uv_count" "ads_uv_count" "dt"
     export_data "ads_user_retention_day_rate" "ads_user_retention_day_rate" "create_date,retention_day"
     export_data "ads_user_topic" "ads_user_topic" "dt"
;;
*)
    echo "参数不对"
    exit;
;;
esac

```

### 参数说明
 关于导出update还是insert的问题：
```
--update-mode：
    updateonly   只更新，无法插入新数据
    allowinsert   允许新增 

-update-key：允许更新的情况下，指定哪些字段匹配视为同一条数据，进行更新而不增加。多个字段用逗号分隔。
```

空值处理：
```
--input-null-string和--input-null-non-string：
分别表示，将字符串列和非字符串列的空串和“null”转义。
```
Hive中的Null在底层是以“\N”来存储，而MySQL中的Null在底层就是Null，为了保证数据两端的一致性。在导出数据时采用--input-null-string和--input-null-non-string两个参数。导入数据时采用--null-string和--null-non-string。

### 脚本测试
脚本可以使用
```
[hadoop@hadoop101 sqoop]$ ./sqoop_export.sh all
```

## 任务调度
需要调度的脚本：
```sh
# 用sqoop把mysql导入数据到hdfs
mysql_to_hdfs.sh all 2020-03-11

# hdfs数据剪切到hive的ods层
hdfs_to_ods_log.sh  2020-03-11
hdfs_to_ods_db.sh all 2020-03-11

# ods层加载到dwd
ods_to_dwd_start_log.sh 2020-03-11
ods_to_dwd_base_event_log.sh 2020-03-11
ods_to_dwd_detail_event_log.sh 2020-03-11
ods_to_dwd_db.sh 2020-03-11

# dwd到dws
dwd_to_dws.sh 2020-03-11
# dws到dwt
dws_to_dwt.sh 2020-03-11

# ads层出统计结果
dws_dwt_to_ads.sh 2020-03-11

# 用sqoop把ads层统计结果导出到mysql
sqoop_export.sh all
```

创建各个job：
```sh
lifengchao@lifengchao-YangTianM4601c-00:~/codes/ecdwFlow$ ll
总用量 52
drwxr-xr-x 2 lifengchao lifengchao 4096 7月  21 13:46 ./
drwxr-xr-x 7 lifengchao lifengchao 4096 7月  21 12:41 ../
-rw-r--r-- 1 lifengchao lifengchao  274 7月  21 14:04 ads_to_mysql.job
-rw-r--r-- 1 lifengchao lifengchao  327 7月  21 14:01 dwd_to_dws.job
-rw-r--r-- 1 lifengchao lifengchao  283 7月  21 14:03 dws_dwt_to_ads.job
-rw-r--r-- 1 lifengchao lifengchao  275 7月  21 14:02 dws_to_dwt.job
-rw-r--r-- 1 lifengchao lifengchao  322 7月  21 13:55 hdfs_to_ods_db.job
-rw-r--r-- 1 lifengchao lifengchao  262 7月  21 13:51 hdfs_to_ods_log.job
-rw-r--r-- 1 lifengchao lifengchao  252 7月  21 13:43 mysql_to_hdfs.job
-rw-r--r-- 1 lifengchao lifengchao  311 7月  21 13:58 ods_to_dwd_base_event_log.job
-rw-r--r-- 1 lifengchao lifengchao  285 7月  21 13:58 ods_to_dwd_db.job
-rw-r--r-- 1 lifengchao lifengchao  325 7月  21 13:59 ods_to_dwd_detail_event_log.job
-rw-r--r-- 1 lifengchao lifengchao  334 7月  21 13:55 ods_to_dwd_start_log.job
```

打包：
```sh
zip ecdwFlowJobs.zip *.job
```


```sql

```