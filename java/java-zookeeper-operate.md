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

zkCli客户端也提供了丰富的操作命令
```
$ bin\zkCli.sh -server 127.0.0.1:2181
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

node有几个版本号：
```
cversion = 4	# 子版本号
dataVersion = 0	#数据版本号
aclVersion = 0	#权限版本号
```

权限
zookeeper中权限通常表示为scheme:id:permissions
（1）scheme: scheme对应于采用哪种方案来进行权限管理，zookeeper实现了一个pluggable的ACL方案，可以通过扩展scheme，来扩展ACL的机制。zookeeper-3.4.4缺省支持下面几种scheme:

    world: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的
    auth: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)
    digest: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
    ip: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段
    super: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

另外，zookeeper-3.4.4的代码中还提供了对sasl的支持，不过缺省是没有开启的，需要配置才能启用，具体怎么配置在下文中介绍。

    sasl: sasl的对应的id，是一个通过sasl authentication用户的id，zookeeper-3.4.4中的sasl authentication是通过kerberos来实现的，也就是说用户只有通过了kerberos认证，才能访问它有权限的node.


id: id与scheme是紧密相关的，具体的情况在上面介绍scheme的过程都已介绍，这里不再赘述。

permission: zookeeper目前支持下面一些权限：

    CREATE(c): 创建权限，可以在在当前node下创建child node
    DELETE(d): 删除权限，可以删除当前的node下的child node
    READ(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes
    WRITE(w): 写权限，可以向当前node写数据
    ADMIN(a): 管理权限，可以设置当前node的permission


java客户端
zookeeper提供了java客户端

```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```
 举个栗子
```
        Watcher basicWatcher = new Watcher() {
            public void process(WatchedEvent event) {
                System.out.println(event.getPath() + " processed event : " + event.getType() );
            }
        };

        // 连接ZooKeeper
        ZooKeeper zk = new ZooKeeper("localhost:2181",5000, basicWatcher);


        // 权限管理
        List<ACL> acls = new ArrayList<ACL>();
        Id id = new Id("digest",DigestAuthenticationProvider.generateDigest("admin:admin"));
        acls.add(new ACL(ZooDefs.Perms.ALL, id));
        // 为会话添加身份
        zk.addAuthInfo("digest", "admin:admin".getBytes());

        // 创建一个目录节点
        zk.create("/hello", "hello,zookeeper".getBytes(), acls,
                CreateMode.PERSISTENT);



        // 获取子node，最后一个参数为true，将使用basicWatcher监听/hello
        zk.getChildren("/hello", true);

        // 创建一个子目录节点
        zk.create("/hello/node1", null,
                ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

        // 修改子目录节点数据,最后一个参数为dataVersion，在并发情况下可以实现乐观锁，用-1匹配所有版本
        zk.setData("/hello/node1", "first".getBytes(), 0);
        zk.setData("/hello/node1", "second".getBytes(), 1);

        // 删除node
        zk.delete("/hello/node1", 2);
        zk.delete("/hello", 0);
```

参考：
http://www.wuzesheng.com/?p=2438