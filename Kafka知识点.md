## Kafka知识点

### kafka特性

```java
1.高吞吐量 2.扩展性 3.持久化（消息持久化磁盘） 4.容错性 （多个broker 搭建成集群）5.高并发
```

### Kafka设计思想

```java
基于zk的选举策略
    1.所有节点一起去zk上注册一个临时节点，但是只能有一个注册成功，注册成功的为kafka的leader,其余的为follower。
    2.leader监听 所有broker的消息，当leader宕机，zk上的临时节点就会消失，其他的节点再次一起去zk上注册节点，又会只有一个注册成功，这个成为新的leader
    
```

### Consumer Group



### Producer

