## 初体验
1.[官网](http://zookeeper.apache.org/)下载，解压
```
# wget -c http://mirrors.cnnic.cn/apache/zookeeper/stable/zookeeper-3.4.9.tar.gz
# tar -xvz -f zookeeper-3.4.9.tar.gz
# cd zookeeper-3.4.9
```
我下载的是最新的稳定版本3.4.9

2.添加配置  
conf目录下默认提供了一个简单的配置文件zoo_sample.cfg，创建文件conf/zoo.cfg，内容为
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/my/zookeeper-3.4.9/serviceData
clientPort=2181
```
注意先创建dataDir

3.启动
```
# bin/zkServer.sh start
# jps
5156 QuorumPeerMain
6120 Jps
```
默认使用conf/zoo.cfg作为配置文件启动  
jps可以看到QuorumPeerMain进程正在运行。

4.连接
```
# bin/zkCli.sh -server 127.0.0.1:2181
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 2] quit	
#
```

5.关闭

```
# bin/zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /usr/local/my/zookeeper-3.4.9/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

## 单机伪集群配置  
1. 创建dataDir：server0和server1，这两个目录下创建myid文件，内容分别为0,1
```
# cat server0/myid
0
# cat server1/myid
1
```
myid中的数字表示服务器id


2. 创建两个配置文件conf/zoo0.cfg和conf/zoo1.cfg，内容为

```
#conf/zoo0.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/my/zookeeper-3.4.9/server0
clientPort=2180

server.0=127.0.0.1:2880:3880
server.1=127.0.0.1:2881:3881

#conf/zoo1.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/my/zookeeper-3.4.9/server1
clientPort=2181

server.0=127.0.0.1:2880:3880
server.1=127.0.0.1:2881:3881
```
server.A=B：C：D:其中 A 是一个数字，就是myid里的那个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址,C和D是两个端口。


两个端口的作用，官网描述如下：


>Finally, note the two port numbers after each server name: “ 2888” and “3888”. Peers use the former port to connect to other peers. Such a connection is necessary so that peers can communicate, for example, to agree upon the order of updates. More specifically, a ZooKeeper server uses this port to connect followers to the leader. When a new leader arises, a follower opens a TCP connection to the leader using this port. Because the default leader election also uses TCP, we currently require another port for leader election. This is the second port in the server entry.


简单来说，第一个端口用来集群成员的信息交换以及与集群中的Leader 服务器交换信息，第二个端口是在leader挂掉时专门用来进行选举leader所用。

因为是伪分布式，所以dataDir,clientPort也不一样，同时C,D两个端口也不能相同。

启动
```
# bin/zkServer.sh start zoo0.cfg
# bin/zkServer.sh start zoo1.cfg
# jps
17759 Jps
17701 QuorumPeerMain
17664 QuorumPeerMain
```

可以看到两个QuorumPeerMain进程进行运行

查看节点状态
```
# bin/zkServer.sh status zoo0.cfg
ZooKeeper JMX enabled by default
Using config: /usr/local/my/zookeeper-3.4.9/bin/../conf/zoo0.cfg
Mode: follower
# bin/zkServer.sh status zoo1.cfg
ZooKeeper JMX enabled by default
Using config: /usr/local/my/zookeeper-3.4.9/bin/../conf/zoo1.cfg
Mode: leader
```

同步操作
```
# bin/zkCli.sh -server 127.0.0.1:2180
[zk: 127.0.0.1:2180(CONNECTED) 0] ls /
[zookeeper]
[zk: 127.0.0.1:2180(CONNECTED) 1] ls
[zk: 127.0.0.1:2180(CONNECTED) 2] create /hello "hello"
Created /hello
[zk: 127.0.0.1:2180(CONNECTED) 3] ls /
[hello, zookeeper]
[zk: 127.0.0.1:2180(CONNECTED) 4] quit
Quitting...
# bin/zkCli.sh -server 127.0.0.1:2181
[zk: 127.0.0.1:2181(CONNECTED) 0] ls /
[hello, zookeeper]
```
可以看到主从同步成功。

参考：  
https://my.oschina.net/vbird/blog/384043  
http://brianway.github.io/2016/06/29/zookeeper-replicated/