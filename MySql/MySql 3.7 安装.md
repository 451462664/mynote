# MySql 安装

## 下载

直接去官方网站下载需要的版本即可。

## 解压

将下载的文件解压到自己喜欢的位置，与 MySql 5.6 不一样的是 5.7 版本中没有 data 文件夹和 my-default.ini 文件。

## 配置

在 MySql 文件夹新建 my.ini 文件

```ini
[mysqld] 
#设置3306端
port = 3306
# 设置mysql的安装目录
basedir=D:\mysql-5.7.27
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql-5.7.27\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
 
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
```

## 安装 MySql 服务

以管理员身份运行 cmd 将目录设置到 MySql 的 bin 目录下。执行 mysqld install。

## 初始化 MySql

继续在 bin 目录下继续输入 mysqld --initialize --user root --console 然后会在下面告诉我们初始密码。

## 启动 MySql

继续在 bin 目录下执行 net start mysql 启动服务。

## 修改密码

继续在 bin 目录下执行 mysql -u root -p  输入刚才的密码登录。
然后通过 set password=password('123456') 来修改密码。

## 关闭 MySql 服务

以管理员身份运行 cmd 执行命令 net stop mysql。

## 移除 MySql 服务

mysqld -remove [服务名]