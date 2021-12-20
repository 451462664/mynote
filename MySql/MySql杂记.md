```mysql
# 查看数据库连接池状态
SHOW PROCESSLIST ;

# 查看 binlog
SHOW BINARY LOGS;

SET tx_isolation='READ-COMMITTED';
# 如果选择global，意思是此语句将应用于之后的所有session，而当前已经存在的session不受影响。
# 如果选择session，意思是此语句将应用于当前session内之后的所有事务。
# 如果什么都不写，意思是此语句将应用于当前session内的下一个还未开始的事务。
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
SHOW VARIABLES LIKE 'transaction_isolation';

# 查看长事务
SELECT * FROM information_schema.innodb_trx WHERE TIME_TO_SEC(TIMEDIFF(NOW(),trx_started))>60;


SELECT * 
FROM BASKETBALL_COURT
WHERE PLAYER BETWEEN 0 AND 12
AND LIGHT IS NOT NULL;

+ - - - + - - - +
| NAME  |PLAYER |
+ - - - + - - - +
| NULL  | NULL  |
+ - - - + - - - +

# order by
select * from product order by 字段A desc,字段B asc;
影响：数据会先按照第一个字段排序（price）,如果第一个字段的值相同，再按照第二个字段排序！

# group by
select * from product group by 字段A, 字段B;
GROUP BY X 意思是将所有具有相同 X 字段值的记录放到一个分组里，GROUP BY X, Y意思是将所有具有相同X字段值和Y字段值的记录放到一个分组里。
```

