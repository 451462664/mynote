```mysql
# 查看数据库连接池状态
SHOW PROCESSLIST ;

# LOAD-DATA 导入 csv 数据
LOAD DATA LOCAL INFILE '/Users/1.csv'
INTO TABLE safety_center.oil_compensation
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1
LINES(tid, gpsdate, longitude, latitude, speed, standardmileage, receivedate, ispatch, oilvalue);

# order by
select * from product order by 字段A desc,字段B asc;
影响：数据会先按照第一个字段排序（price）,如果第一个字段的值相同，再按照第二个字段排序！

# group by
select * from product group by 字段A, 字段B;
GROUP BY X 意思是将所有具有相同 X 字段值的记录放到一个分组里，GROUP BY X, Y意思是将所有具有相同X字段值和Y字段值的记录放到一个分组里。

# mysql dump 导出
/usr/local/bin/mysqldump -uzhdbuser -h10.60.247.10 -P3309 -p72kVFvXvCpWI76oHQIGJsRw6 --single-transaction --set-gtid-purged=OFF smart_team b_day_statistics_2021_10_temp > /Users/wmerake/Desktop/1.sql

# mysql dump 跨服务器复制库
mysqldump smart_team -u zhdbuser -p72kVFvXvCpWI76oHQIGJsRw6 --add-drop-table | mysql smart_team -h 172.29.30.96 -u zhdbuser -p72kVFvXvCpWI76oHQIGJsRw6
```

