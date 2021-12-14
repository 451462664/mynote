```sql
# order by
select * from product order by 字段A desc,字段B asc;
影响：数据会先按照第一个字段排序（price）,如果第一个字段的值相同，再按照第二个字段排序！

# group by
select * from product group by 字段A, 字段B;
GROUP BY X 意思是将所有具有相同 X 字段值的记录放到一个分组里，GROUP BY X, Y意思是将所有具有相同X字段值和Y字段值的记录放到一个分组里。
```

# MySQL

## 重新认识 MySQL

日常使用 MySQL 一般是这样的

1. 启动 MySQL 服务器程序
2. 启动 MySQL 客户端程序并连接到服务器程序
3. 在客户端程序中输入一些命令语句作为请求发送到服务器程序，服务器程序收到这些请求后，会根据请求的内容来操作具体的数据并向客户端返回操作结果

### 启动 MySQL 服务器程序

#### MySQL bin 目录下可执行文件说明

1. mysqlId: 这个可执行文件代表着 MySQL 服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。
2. mysqlId_safe: 是一个启动脚本，它会间接的调用 mysqlId，而且还顺便启动了另外一个监控进程，这个监控进程在服务器进程挂了的时候，可以帮助重启它。另外，使用 mysqlId_safe 启动服务程序时，它会将服务器程序的出错信息和其他诊断信息重定向到某个文件中，产生出错日志，这样可以方便我们找出错误的原因。
3. mysql.server: 也是一个启动脚本，它会间接的调用 mysqlId_safe 在调用 mysql.server 时在后边指定 start 参数就可以启动服务程序了，比如 `mysql.server start`。stop 可以关闭正在运行的服务器程序。
4. mysqlid_multi: 其实我们一台计算机也可以运行多个服务器实例，也就是运行多个 MySQL 服务器进程。

### 启动 MySQL 客户端程序

bin 目录下有许多客户端程序，比方说 mysqladmin、mysqldump、mysqlcheck 等等。我们只需要重点关注 mysql，通过这个命令可以让我们和服务器进程交互，也就是发送请求接收服务端的处理结果。

`mysql -h主机名 -u用户名 -p密码`

### 服务器处理客户端请求

> 客户端进程向服务器进程发送一段文本(MySQL 语句)，服务器进程处理后再向客户端进程发送一段文本(处理结果)。



