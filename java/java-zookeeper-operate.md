ZooKeeper 支持某些特定的命令与linux进行交互。它们大多是查询命令，用来获取 ZooKeeper 服务的当前状态及相关信息。

ZooKeeper 四字命令 |  功能描述 
---|---
conf | 输出相关服务配置的详细信息。
cons | 列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。包括“接受 / 发送”的包数量、会话 id 、操作延迟、最后的操作执行等等信息。
dump | 列出未经处理的会话和临时节点。
envi | 输出关于服务环境的详细信息（区别于 conf 命令）。
reqs | 列出未经处理的请求
ruok | 测试服务是否处于正确状态。如果确实如此，那么服务返回“ imok ”，否则不做任何相应。
stat | 输出关于性能和连接的客户端的列表。
wchs | 列出服务器 watch 的详细信息。
wchc | 通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
wchp | 通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。 

zookeeper也提供了客户端
```
$ bin\zkCli.cmd -server 127.0.0.1:2181
[zk: 127.0.0.1:2181(CONNECTED) 1] help
ZooKeeper -server host:port cmd args
        stat path [watch]	# 获取znode属性，并且监听其是否存在
        set path data [version]	# 设置znode数据
        ls path [watch] # 查看子znode
        delquota [-n|-b] path # 删除quota设置
        ls2 path [watch] # 查看znode详细信息
        setAcl path acl	# 设置权限
        setquota -n|-b val path	# 设置quota配置
        history	# 查看历史
        redo cmdno	# 重做命令
        printwatches on|off	# 
        delete path [version] #删除znode
        sync path # 同步znode
        listquota path	
        rmr path # 递归删除节点及子znode
        get path [watch] # 
        create [-s] [-e] path data acl	# 创建znode
        addauth scheme auth	# 
        quit
        getAcl path
        close
        connect host:port
```

znode