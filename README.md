# Consul 简介、安装、常用命令的使用

## 1 Consul简介

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其他分布式服务注册与发现的方案，Consul的方案更“一站式”，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value存储、多数据中心方案，不再需要依赖其他工具（比如ZooKeeper等）。使用起来也较 为简单。Consul使用Go语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与Docker等轻量级容器可无缝配合 。

## 2 Consul安装

从consul官网: https://www.consul.io/downloads.html ,进行下载就好,解压consul_0.6.4_darwin_amd64.zip,将解压后的二进制文件consul（拷贝到/usr/local/bin下）; sudo scp consul /usr/local/bin

## 3 Consul 启动

./consul agent -dev              # -dev表示开发模式运行，另外还有-server表示服务模式运行
consul agent -dev -client 127.0.0.1

说明：
-dev（该节点的启动不能用于生产环境，因为该模式下不会持久化任何状态），该启动模式仅仅是为了快速便捷的启动单节点consul
* 该节点处于server模式 
* 该节点是leader 
* 该节点是一个健康节点 

consul members   #查看consul cluster中的每一个consul节点的信息

说明：
* Address：节点地址
* Status：alive表示节点健康
* Type：server运行状态是server状态
* DC：dc1表示该节点属于DataCenter1 

### 4 停止服务（优雅退出）
命令：CTRL+C
说明：
* 该节点离开后，会通知cluster中的其他节点

## 5 Consul常用命令

1. agent	;运行一个consul agent;	consul agent -dev
1. join	;将agent加入到consul集群;	consul join IP
1. members	;列出consul cluster集群中的members;	consul members
1. leave	;将节点移除所在集群;	consul leave

## 6 Consul 的高可用

这边准备了三台CentOS 7的虚拟机，主机规划如下，供参考：

主机名称	IP	作用	是否允许远程访问
* node0	192.168.11.143	consul server	是
* node1	192.168.11.144	consul client	否
* node2	192.168.11.145	consul client	是
### 搭建步骤
1. 启动node0机器上的Consul（node0机器上执行）：
* consul agent -data-dir /tmp/node0 -node=node0 -bind=192.168.11.143 -datacenter=dc1 -ui -client=192.168.11.143 -server -bootstrap-expect 1
1. 启动node1机器上的Consul（node1机器上执行）：
* consul agent -data-dir /tmp/node1 -node=node1 -bind=192.168.11.144 -datacenter=dc1 -ui
1. 启动node2机器上的Consul（node2机器上执行）：
* consul agent -data-dir /tmp/node2 -node=node2 -bind=192.168.11.145 -datacenter=dc1 -ui -client=192.168.11.145
1. 将node1节点加入到node0上（node1机器上执行）：
* consul join 192.168.11.143
1. 将node2节点加入到node0上（node2机器上执行）：
* consul join -rpc-addr=192.168.11.145:8400  192.168.11.143
1. 这样一个简单的Consul集群就搭建完成了，在node1上查看当前集群节点：
* consul members -rpc-addr=192.168.11.143:8400

结果如下：
* node0  192.168.11.143:8301  alive   server  0.7.0   2         dc1
* node1  192.168.11.144:8301  alive   client  0.7.0   2         dc1
* node2  192.168.11.145:8301  alive   client  0.7.0   2         dc1

