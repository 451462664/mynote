# Docker

## 常用命令

### 镜像命令

```shell
docker images # 查看所有本地主机上的镜像
docker search # 在 dockerHub 上搜索镜像
docker pull [镜像名] # 下载镜像
docker rmi [镜像 id] # 删除镜像

# 可选项
-a，--all # 列出所有镜像
-q，--quiet # 只显示镜像的 id

docker images -aq # 显示所有镜像的 id
```

### 容器命令

```shell
docker run [镜像id] # 新建容器并启动
docker ps # 列出所有运行的容器 docker container list
docker rm [容器 id] # 删除指定容器
docker start [容器 id] # 启动指定容器
docker restart [容器 id] # 重启指定容器
docker stop [容器 id] # 停止当前正在运行的容器
docker kill [容器 id] # 强制停止当前容器
```

**我们先有镜像才可以创建容器**

#### 新建容器并启动

```shell
docker run [可选参数] [镜像id]
# 参数说明
--name="Name" 容器名字 tomcat01 tomcat02 用来区分容器
-d 后台方式运行
-it 使用交互方式运行，进入容器查看内容
-p 指定容器的端口 -p 8888:8080
	-p ip:主机端口：容器端口
	-p 主机端口：容器端口
	-p 容器端口
-P（大写）随机指定端口

# 启动并进入容器
docker run -it centos /bin/bash
exit # 从容器中退出（容器直接退出）
ctrl + p + q # 容器不停止退出
```

#### 删除容器

```shell
docker rm [容器 id] # 删除指定容器，不能删除正在运行的容器，如果要强制删除 rm -rf
docker rm -f ${docker ps -aq} # 批量删除指定的容器
```

#### 常用其他容器命令

```shell
docker top [容器 id] # 查看容器中进程信息
docker inspect [容器 id] # 查看容器元数据
docker extc [容器 id] # 进入当前正在运行的容器
docker attach [容器 id] # 进入当前正在运行的容器
# docker exec 进入当前容器后开启一个新的终端，可以在里面操作
# docker attach 进入容器正在执行的终端
```
