
# DWT层

## 设备主题宽表
![](assets/markdown-img-paste-20200714233301201.png)

### 建表语句
```sql
drop table if exists dwt_uv_topic;

create external table dwt_uv_topic
(
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识',
    `version_code` string COMMENT '程序版本号',
    `version_name` string COMMENT '程序版本名',
    `lang` string COMMENT '系统语言',
    `source` string COMMENT '渠道号',
    `os` string COMMENT '安卓系统版本',
    `area` string COMMENT '区域',
    `model` string COMMENT '手机型号',
    `brand` string COMMENT '手机品牌',
    `sdk_version` string COMMENT 'sdkVersion',
    `gmail` string COMMENT 'gmail',
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度',
    `login_date_first` string  comment '首次活跃时间',
    `login_date_last` string  comment '末次活跃时间',
    `login_day_count` bigint comment '当日活跃次数',
    `login_count` bigint comment '累积活跃天数'
)
stored as parquet
location '/warehouse/gmall/dwt/dwt_uv_topic';
```

### 导入数据
```sql
insert overwrite table dwt_uv_topic
select
    -- nvl能实现的，if都能实现，这里就是练一下nvl。
    -- if 和 nvl两个参数的返回类型必须一致：
    -- T if(boolean testCondition, T valueTrue, T valueFalseOrNull)
    -- T nvl(T value, T default_value)
    nvl(new.mid_id, old.mid_id),
    nvl(new.user_id, old.user_id),
    nvl(new.version_code, old.version_code),
    nvl(new.version_name, old.version_name),
    nvl(new.lang, old.lang),
    nvl(new.source, old.source),
    nvl(new.os, old.os),
    nvl(new.area, old.area),
    nvl(new.model, old.model),
    nvl(new.brand, old.brand),
    nvl(new.sdk_version, old.sdk_version),
    nvl(new.gmail, old.gmail),
    nvl(new.height_width, old.height_width),
    nvl(new.app_time, old.app_time),
    nvl(new.network, old.network),
    nvl(new.lng, old.lng),
    nvl(new.lat, old.lat),
    nvl(old.login_date_first, '2020-03-10'),
    if(new.mid_id is not null, '2020-03-10', old.login_date_last),
    nvl(new.login_count, 0),
    nvl(old.login_count, 0) + if(new.login_count > 0, 1, 0)
select(
    select
        *
    from dwt_uv_topic
) old
full join(
    select
        *
    from dws_uv_detail_daycount
    where dt = '2020-03-10'
) new on old.mid_id = new.mid_id;
```

查看：
```sql
select * from dwd_dim_sku_info where dt='2020-03-10' limit 2;
```

## aa

### 建表语句
```sql

```

### 导入数据
```sql

```

查看：
```sql
select * from dwd_dim_sku_info where dt='2020-03-10' limit 2;
```

## aa

### 建表语句
```sql

```

### 导入数据
```sql

```

查看：
```sql
select * from dwd_dim_sku_info where dt='2020-03-10' limit 2;
```









