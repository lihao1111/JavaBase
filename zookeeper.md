## Zk基础知识点

### 分布式CAP

```java
C: 一致性	A：高可用行	P：分区容错性	
ZK使用CP    
```

### 分布式

### zk的选举

```properties
1.假设有三台服务器 zoo.cfg中配置 
server.0=bigdatatopdraw003:2889:3889
server.4=bigdatatopdraw001:2889:3889
server.5=bigdatatopdraw002:2889:3889

2.开启对应的服务器1会在 data目录中生成一个myid 对应的 server的序号（权重）

1.当服务器1 开启的时候，自己给自己投票，然后发投票信息，由于其它机器还没有启动所以它收不到反馈信息，服务器1的状态一直属于Looking(选举状态)。
2.当服务器2 开启的时候，自己给自己投票，同时与之前启动的服务器1交换结果，由于服务器2的编号大所以服务器2胜出，但此时投票数没有大于半数，所以两个服务器的状态依然是LOOKING。
3.当服务器3 开启的时候，自己给自己图片，同时与之前的1,2服务器交换结果，由于服务器2的编号大所以服务器3胜出，并且超过半数，此时服务3为leader；1,2服务器为follower
```

### zk实现分布式锁

```java
同一目录下只能不可以有重名的文件
    1,A线程获取锁，就在目录下创建一个顺序节点，如果是最新的，线程顺序号最小，获得锁；
    2，B线程获取所有节点，判断自己是不是最小的，不是最小的则监听最小的节点；
    3，A释放锁，删除最小节点
    4，B线程获取锁

    
```
