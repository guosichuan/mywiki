---
title: "Q2t"
date: 2016-06-07 11:18
---

#Q2需求
@(bigdata)[hive, mysql]



###来自updateId表
- 各应用分平台用户总数
1. 总数是多长时间内的总数,每个月统计一回

|day| app|     platform|   usr_num|
| :--------| :-------- | --------:| :------: |
| field1 | field1    |   field2 |  field3  |

```sql
select '$STAT_DAY' as day,app,platform,count(1) from (
select upid.app,
CASE
  WHEN upid.idfa is not NULL and upid.idfa != ''  THEN 'ios'
  WHEN upid.imei is not NULL and upid.imei != ''  THEN 'android'
  ELSE 'other'
END as platform from mobile_day_id_lastest upid
 )t group by app,platform order by app,platform
```
```sql
CREATE TABLE `tb_usr_stat` (
  `day` date DEFAULT NULL,
  `app_code` varchar(50) DEFAULT NULL,
  `platform` varchar(20) DEFAULT NULL,
  `usr_num` int(11) DEFAULT NULL
)  ENGINE=MyISAM DEFAULT CHARSET=utf8
```



- 各应用分平台新用户（按日、月、全部），并按地域分布

| day/month/all| app|     platform|   province|   city|   new_usr_num|
| :-------- | :-------- | --------:| :------: |:------: |:------: |
| field0   | field1    |   field2 |  field3  |field4  |field5  |



1. 需要第一次登录时间

```sql
select '2016-06-01' as day ,app,platform,province,city, count(1) from
(select upid.app,
CASE
  WHEN upid.idfa is not NULL and upid.idfa != ''  THEN 'ios'
  WHEN upid.imei is not NULL and upid.imei != ''  THEN 'android'
  ELSE 'other'
END as platform,province,city from mobile_day_id_lastest upid where first_uptime>= 1467334800 and first_uptime < 1464710400 ) t
group by app,platform,province,city
```
```sql
CREATE TABLE `tb_newusr_month` (
  `day` date DEFAULT NULL,
  `app_code` varchar(50) DEFAULT NULL,
  `platform` varchar(20) DEFAULT NULL,
  `province` varchar(50) DEFAULT NULL,
  `city` varchar(50) DEFAULT NULL,
  `new_usr_num` int(11) DEFAULT NULL
)  ENGINE=MyISAM DEFAULT CHARSET=utf8
```
```sql
CREATE TABLE `tb_newusr_day` (
  `day` date DEFAULT NULL,
  `app_code` varchar(50) DEFAULT NULL,
  `platform` varchar(20) DEFAULT NULL,
  `province` varchar(50) DEFAULT NULL,
  `city` varchar(50) DEFAULT NULL,
  `new_usr_num` int(11) DEFAULT NULL
)  ENGINE=MyISAM DEFAULT CHARSET=utf8
```
```sql
CREATE TABLE `tb_newusr` (
  `day` date DEFAULT NULL,
  `app_code` varchar(50) DEFAULT NULL,
  `platform` varchar(20) DEFAULT NULL,
  `province` varchar(50) DEFAULT NULL,
  `city` varchar(50) DEFAULT NULL,
  `new_usr_num` int(11) DEFAULT NULL
)  ENGINE=MyISAM DEFAULT CHARSET=utf8
```



- 各应用分平台苏醒用户
| day| app|     platform|   province|   city|   revive_usr_num|
| :-------- | :-------- | --------:| :------: |:------: |:------: |
| field0   | field1    |   field2 |  field3  |field4  |field5  |
```sql
select app,平台 count(1) from mobile_day_id_lastest where revive_date betweeen(day1,day2) group by app,平台
```

- 有urs账号的用户比例

| month| app|platform|urs_num|
| :-------- | :-------- | :-------- | :-------- | 
| field0   | field1    | field1    | field1    | 


```sql
select app,平台 count(1) from mobile_day_id_lastest where urs is not null group by app,平台
```

- 各应用各个版本的占比

| month| app|platform|appver|usr_num|
| :-------- | :-------- | :-------- | :-------- | 
| field0   | field1    | field1    | field1    | 

```sql
select app,平台,appver,count(1) from mobile_day_id_update group by app,平台,appver
```

15、新闻客户端与其他app用户重合度


###来自Adlog表
- 各应用分平台DAU，

```sql
select day,app_id,platform,count(1) from (select ad.day,ads.app_id,ads.platform,ad.deviceid from mobile_ad_space ads inner join (select day,deviceid,flightid from mobile_day_adlog where day='$STAT_DAY') ad on cast(ads.space_id as string)=ad.flightid   group by ad.day,ads.app_id,ads.platform,ad.deviceid) day_active group by day,app_id,platform order by day,app_id,platform;
```

- 各应用分平台DAU,并按地域分布

```sql
select day,app_id,platform,province,city,count(1) from (select ad.day,ads.app_id,ads.platform,ad.province,ad.city,ad.deviceid from mobile_ad_space ads inner join (select day,deviceid,flightid,province,city from mobile_day_adlog where day='$STAT_DAY') ad on cast(ads.space_id as string)=ad.flightid   group by ad.day,ads.app_id,ads.platform,ad.province,ad.city,ad.deviceid) day_active group by day,app_id,platform,province,city order by day,app_id,platform,province,city;
```

```sql
CREATE TABLE `tb_dau_by_loc` (
  `day` date DEFAULT NULL,
  `appid` int(11) DEFAULT NULL,
  `platform` varchar(20) DEFAULT NULL,
  `province` varchar(50) DEFAULT NULL,
  `city` varchar(50) DEFAULT NULL,
  `dau` int(11) DEFAULT NULL
)
```

```sql
sqoop export --connect jdbc:mysql://$HOST:3306/sqoop --username $USER --password $PASS --table $MYSQL_TABLE  --export-dir /user/mad/mobilead/hql_output/dau_by_loc/$STAT_DAY --fields-terminated-by '\0001'
```

- 各应用分平台MAU

```sql
select '2016-06-01' as day,app_id,platform,count(1) from (select ads.app_id,ads.platform,ad.deviceid from mobile_ad_space ads inner join (select deviceid,flightid from mobile_day_adlog where day>='2016-05-01' and day<'2016-06-01') ad on cast(ads.space_id as string)=ad.flightid   group by ads.app_id,ads.platform,ad.deviceid) month_active group by app_id,platform order by app_id,platform;
```

- 新闻客户端启动、左滑、封面故事
```sql
select flightid,tag,count(1) from mobile_day_adlog where urs is not null and day='' and app=133 group by flightid,tag
```

```sql
select day,app_id,platform,tag,count(1) from (select ad.day,ads.app_id,ads.platform,ad.tag from mobile_ad_space ads inner join (select day,flightid,tag from mobile_day_adlog where day='2016-05-31') ad on cast(ads.space_id as string)=ad.flightid) day_num group by day,app_id,platform,tag order by day,app_id,platform,tag;
```

-文章页图文用户真正看到的比例

```sql
select flightid,count(1) from mobile_day_adlog where seen is not null and day='' and app=133 group by flightid
```


13、单用户各栏目刷新次数 各栏目第四条信息流去重曝光/各栏目刷新次数，
- refreshtime字段意义？
- 如何统计的刷新次数和去重曝光？



###验证daid是否能在两个应用通用
select * from (select daid,count(1) as c from (select daid,app from mobile_day_id_lastest group by daid,app) t group by daid) t1 where c>1;

没有搜索结果，证明**daid在不同应用中不一样**