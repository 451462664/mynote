# k8s 集群

## 搭建 k8s 环境平台规划

### 单 master node 集群

master 一台机器管理多台 node 节点机器，缺点很明显，如果 master 挂掉就会产生问题，所以生产一般采用多 master 集群。

### 多 master node 集群

在单 master 基础上增加多个 master 进行管理。

## 服务器硬件配置要求

最低要求

> 测试环境
`master`: 2核 4G 20G
`node`: 4核 8G 40G

> 生产环境
`master`、`node`: 更高要求

## 搭建 k8s 集群部署方式

### kubeadm

kubeadm 是一个 k8s 部署工具，提供 kubeadm init 和 kubeadm join 用于快速部署 kubernetes 集群。

### 二进制包

从 github 下载发行版的二进制包，手动部署每个组件，组成 kubernetes 集群。

kubeadm 价格低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署 kubernetes 集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。

#### 二进制包部署步骤

1. 创建多台虚拟机，安装 Linux 操作系统。
2. 操作系统初始化。
    1. 关闭防火墙。
    2. 关闭 selinux。
    3. 关闭 swap。
    4. 根据规划设置主机名。
    5. 在 master 添加 hosts。
    6. 设置时间同步。
3. 为 etcd 和 apiserver 自签证书。
4. 部署 etcd 集群。
5. 部署 master 组件。
    1. master 组件包含 apiserver、controller、scheduler、etcd。
6. 部署 node 组件。
    1. node 组件包含 kubelet、kube-proxy、docker、etcd。
7. 部署集群网络。