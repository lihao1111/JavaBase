### HashMap

![avatar](\images\image-20200109133103500.png)

```java
1.主干结构 Node<K,V>[] table
Node<K,V> implement Map.Entry<K,V>	//Node点
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;    //用来定位数组索引位置
    final K key;
    V value;
    Node<K,V> next;   //链表的下一个node

    Node(int hash, K key, V value, Node<K,V> next) { ... }
    public final K getKey(){ ... }
    public final V getValue() { ... }
    public final String toString() { ... }
    public final int hashCode() { ... }
    public final V setValue(V newValue) { ... }
    public final boolean equals(Object o) { ... }
}
2.HashMap哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，主要是为了在取模和扩容时做优化，同时减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。
3.而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能
```

### 为什么使用到了红黑树

```java
1.在CurrentHashMap中是加锁了的，实际上是读写锁，如果写冲突就会等待，
如果插入时间过长必然等待时间更长，而红黑树相对AVL树他的插入更快！
2.红黑树和AVL树都是最常用的平衡二叉搜索树，它们的查找、删除、修改都是O(lgn) time。
```

### HashMap  tableSizeFor () 

```java
根据传入初始值参数 创建Map
tableSizeFor(int cap)				   //通过一个初始容量值 创建map 返回一个等于大于 cap 并且最接近cap 并且是2的次幂的数   
//cap 传入的容量参数  
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
    	//经过上述 n|=n-1等五步，该算法让最高位的1后面的位全变为1。例如 01000 -> 01111.
    	// n = cap-1 避免 cap =2次幂造成的问题，例如 cap =8 的时候 返回的就是 16，我们想要的其实是8.
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
```



HashMap 计算 table[] 的索引位置

```java
HashMap 计算 table[index]
// 方法一，jdk1.8 & jdk1.7都有：
static final int hash(Object key) {
     int h;
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 方法二，jdk1.7有，jdk1.8没有这个方法，但是实现原理一样的：
static int indexFor(int h, int length) {
     return h & (length-1);   //aka  h % length
}
三步原则
    (1) 取key的hashCode值，h = key.hashCode()
	(2) 高位参与运算，h ^ (h >>> 16)
	(3) 取模运算，h & (length-1)
当length总是2的n次方时，h & (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。
```

### HashMap 的 put(K,V)方法

![avatar](\images\image-20200109153054417.png)

```
1.判断 table.length == 0,如果为0 进行 resize()扩容；
2.如果 table不为空数组，如果 key 进行 getIndex(key) 找到所在数组的索引位置，如果 table[index] == null,直接插入；如果key不存在，则判断 tbale[index]是否是treeNode 结构，如果是treeNode，则插入<K,V> 然后进行balanceTreeNode 平衡二叉树；如果不是treeNode，插入键值对 判断链表长度是否 >8 如果 >8，则treeify() 链表转为红黑树，如果插入的时候 key有一样的，则覆盖原来的value，并将原来的value return；最后modCount++，判断新增元素后的table[]的长度是否 > threshold [是否需要扩容]。
```

### HashMap 的扩容 resize()

![avatar](\images\image-20200109154713951.png)

```java
resize()  //扩容
    1.如果 table[]为null的时候 默认设置长度为16的初始数组，threshold=16 * 0.75;如果table有Node，则设置长度为 16 << 1 == 16 * 2, threshold = 12 << 1 == 12 * 2。
    2.oldTable里面的数据复制到新的table中，其中有三种情况，node节点是一个单节点，没有形成链表，直接复制 newTab[e.hash & (newTabLength -1)]  = e; 
	3.node.next有节点，说明node已经形成一个链表结构或者是红黑树结构，当e instanceof TreeNode，是一个树形结构的话，会通过 (e.hash & oldTabLength) 判断 是在老的索引还是新的索引，如果(e.hash & oldTabLength) ==0 则e 在老的索引，如果(e.hash & oldTabLength) ==1 则 e在新的索引，新的索引=[e.hash & (oldLength - 1) + oldLength]， TreeNode<K,V> loHead = null, loTail = null;TreeNode<K,V> hiHead = null, hiTail = null; 分别存放 (e.hash & bit) == 0) 索引不改变的node节点，以及(e.hash & bit) == 1 新索引的node节点，然后再次通过 >UNTREEIFY_THRESHOLD（6）去判断是否需要继续构建成树形结构，如果节点数 >UNTREEIFY_THRESHOLD（6） 则通过treeify 构建成树形结构，如果小于 6， 则通过 untreeify 树形节点转为list链表。
    重点： 使用 (e.hash & oldTabLength == 0) 来判断是否需要在新的数组中改变索引位置 e.hash & oldTabLength == 1 需要改变索引位置，新的索引为： oldIndex+oldLength。
          都是内存指向的改变
          使用loHead loTail 高低位来进行复制Node
```

### jdk1.7 hashMap扩容的问题

```java
1.如果在并发环境中使用HashMap保存数据，有可能会产生死循环的问题，造成cpu的使用率飙升
 resize() ->  transfer(newTable, initHashSeedAsNeeded(newCapacity));
    entry.netx (1) -> e;
	e -> entry.next; 引用死循环  环形链表
    put时会对容量进行检查。如果在扩容是链表中产生一个环形链表，那么在使用get(...)获取数据时将可能产生死循环。   
```

#### 关键值

```java
DEFAULT_INITIAL_CAPACITY  =  1<< 4 		//aka 16  HashMap的默认数组大小
threshold							    //下一次HashMap扩容的临界值
tableSizeFor(int cap)				    //通过一个初始容量值 创建map 返回一个等于大于 cap 并且最接近cap 并且是2的次幂的数   
    	
       static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
    	//经过上述 n|=n-1等五部，该算法让最高位的1后面的位全变为1。例如 01000 -> 01111.
    	// n = cap-1 避免 cap =2次幂造成的问题，例如 cap =8 的时候 返回的就是 16，我们想要的其实是8.
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
 位运算：
     8421 原则
     1111
     2的幂数 10000 原则
     n << 1 == n *2
     n >> 1 == n /2
     


treeify()	// 构建红黑树。
    
    如果 node元素是一个节点，node.next==null，没有形成链表或者tree,则newTab[e.hash & (newTabLength -1)]  = e;如果 node是一个tree
```

```HTML
(1) 扩容是一个特别耗性能的操作，所以使用HashMap的时候，估算map的大小，初始化的时候给一个大致的数值，避免map进行频繁的扩容。

(2) HashMap是线程不安全的，在并发的环境中建议使用ConcurrentHashMap。 Collections.SynchronizedMap(Map)返回一个同步的map

(3) JDK1.8引入红黑树大程度优化了HashMap的性能，这主要体现在hash算法不均匀时，即产生的链表非常长，这时把链表转为红黑树可以将复杂度从O(n)降到O(logn)。

（4）HashMap是如何工作的？面试时可以这么回答： 
HashMap在Map.Entry静态内部类实现中存储key-value对。HashMap使用哈希算法，在put和get方法中，它使用hashCode()和equals()方法。当我们通过传递key-value对调用put方法的时候，HashMap使用Key hashCode()和哈希算法来找出存储key-value对的索引。Entry存储在LinkedList中，所以如果存在entry，它使用equals()方法来检查传递的key是否已经存在，如果存在，它会覆盖value，如果不存在，它会创建一个新的entry然后保存。当我们通过传递key调用get方法时，它再次使用hashCode()来找到数组中的索引，然后使用equals()方法找出正确的Entry，然后返回它的值
```

### Map为什么使用红黑树

```java
	1. 如果插入一个node引起了树的不平衡，AVL和RB-Tree都是最多只需要2次旋转操作，即两者都是O(1)；但是在删除node引起树的不平衡时，最坏情况下，AVL需要维护从被删node到root这条路径上所有node的平衡性，因此需要旋转的量级O(logN)，而RB-Tree最多只需3次旋转，只需要O(1)的复杂度。
    2. 其次，AVL的结构相较RB-Tree来说更为平衡，在插入和删除node更容易引起Tree的unbalance，因此在大量数据需要插入或者删除时，AVL需要rebalance的频率会更高。因此，RB-Tree在需要大量插入和删除node的场景下，效率更高。自然，由于AVL高度平衡，因此AVL的search效率更高。
    3. map的实现只是折衷了两者在search、insert以及delete下的效率。总体来说，RB-tree的统计性能是高于AVL的

```

#### 红黑树的结构

```java
1.根节点必须是黑色
2.每个节点或者是黑色，或者是红色；同一层级 要么是黑 要么是 红
3.每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
4.如果一个节点是红色的，则它的子节点必须是黑色的。
5.从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。    
```

### ConcurrentHashMap

```java
1.抛弃Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性
2.使用synchronized锁住f元素（链表/红黑树的头元素
3.CAS操作	
    tabAt()该方法用来获取table数组中索引为i的Node元素。
	casTabAt()利用CAS操作设置table数组中索引为i的元素
	setTabAt()该方法用来设置table数组中索引为i的元素

```

### 一致性hash

```java
对 2^32次数取模
    hash(服务器的IP地址) % 2^32
缓存图片
    hash(图片名称) % 2^32
 图片到底缓存在那台服务器
    顺时针方向遇到的第一台服务器就是要缓存的服务器地址
解决hash 环倾斜问题
    实际节点复制为 虚拟节点
优点：
    增减集群的缓存服务器时，只有少量的缓存会失效，回源量较小。
```



















