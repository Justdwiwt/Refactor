# Redis 面试汇总

## Redis 是什么？

Redis 是一个开源（BSD 许可）的，**内存**中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。

## Redis 有什么特点

Redis 是一个 Key-Value 类型的内存数据库，和 memcached 有点像，整个数据库都是在**内存**当中进行加载操作，定期通过异步操作把数据库数据`flush`到硬盘上进行保存。

Redis 的性能很高，可以处理超过 10 万次/秒的读写操作，是目前已知性能最快的 Key-Value DB。

Redis 支持保存多种数据结构，单个 value 的最大限制是 1GB，比 memcached 的 1MB 高。

Redis 可以用来实现很多有用的功能，比方说用他的**List 来做 FIFO 双向链表**，实现一个轻量级的高性能消息队列服务，用他的**Set 可以做高性能的 tag 系统**等等。

Redis 可以对存入的 Key-Value 设置 expire 时间，因此也可以被当作一个功能加强版的 memcached 来用。

Redis 也有缺陷，那就是是**数据库容量受到物理内存的限制**，**不能用作海量数据的高性能读写**，因此 Redis 比较适合那些局限在**较小数据量的高性能操作和运算上**。

## 使用 Redis 有哪些好处？

1. 速度快，因为数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是**O(1)**
2. 支持丰富数据类型，支持`string`，`list`，`set`，`sorted set`，`hash`
3. 支持**事务**，操作都是**原子性**，所谓的原子性就是*对数据的更改要么全部执行，要么全部不执行*
4. 丰富的特性：可用于缓存，消息，按 key 设置**过期时间**，过期后将会自动删除

## 缓存

### 缓存穿透

> 指查询一个一定不存在的数据，因为缓存中也无该数据信息，请求会直接到达数据库，在数据库中查询。

从系统层面来看像是穿透了缓存层直接到达数据库。

没有了缓存层的保护，如果有人频繁的查询这种一定不存在的数据来攻击系统，就会引起数据库瘫痪，从而导致系统故障。

**解决方案**

1. **bloom filter**：类似于哈希表的一种算法，在进行数据库查询之前，进行过滤操作，**对不存在的数据直接过滤**，从而减轻数据库层面的压力。
2. **空值缓存**：将在第一次查询不到的数据（该 key 与对应的空值）放入缓存，并设置短的失效时间，可以在一定时间内，应对攻击者的攻击。

### 缓存雪崩

> 所有的缓存在同一时间失效，系统所有的请求会直接到达数据库，数据库无法承受压力，会导致系统奔溃。

**解决方案：**

1. **线程互斥**：每个时刻**只让一个线程构建缓存**，其他线程等待执行，从而减轻数据库的压力。缺点是**降低了系统的 qps**（每秒处理完请求的次数）。
2. **交错失效时间**：在某个适当的值域随机一个时间作为失效时间。

### 缓存击穿

> 缓存击穿是缓存雪崩的一个特例。缓存击穿是对于热点数据，雪崩是全部数据。

**解决方案：**

1. **二级缓存**：对于热点数据进行二级缓存，对于不同级别的缓存设置不同的实效时间。
2. **LRU**

```java
import java.util.HashMap;

/**
 * LRU算法
 *
 * @author Justdwiwt
 */
public class LRUCache {
    private Node head;
    private Node end;
    //缓存存储上限
    private int limit;

    private HashMap<String, Node> hashMap;

    private LRUCache(int limit) {
        this.limit = limit;
        hashMap = new HashMap<>();
    }

    private String get(String key) {
        Node node = hashMap.get(key);
        if (node == null) return null;
        refreshNode(node);
        return node.value;
    }


    private void put(String key, String value) {
        Node node = hashMap.get(key);
        if (node == null) {
            if (hashMap.size() >= limit) {
                String oldKey = removeNode(head);
                hashMap.remove(oldKey);
            }
            node = new Node(key, value);
            addNode(node);
            hashMap.put(key, node);
        } else {
            node.value = value;
            refreshNode(node);
        }
    }

    @SuppressWarnings("unused")
    public void remove(String key) {
        Node node = hashMap.get(key);
        removeNode(node);
        hashMap.remove(key);
    }

    /**
     * 刷新被访问的节点位置
     *
     * @param node 被访问的节点
     */
    private void refreshNode(Node node) {
        if (node == end) return;
        removeNode(node);
        addNode(node);
    }

    /**
     * 删除节点
     *
     * @param node 要删除的节点
     * @return 删除结点的值
     */
    private String removeNode(Node node) {
        if (node == end)
            end = end.pre;
        else if (node == head)
            head = head.next;
        else {
            node.pre.next = node.next;
            node.next.pre = node.pre;
        }
        return node.key;
    }

    /**
     * 尾部插入节点
     *
     * @param node 要插入的节点
     */
    private void addNode(Node node) {
        if (end != null) {
            end.next = node;
            node.pre = end;
            node.next = null;
        }
        end = node;
        if (head == null) head = node;
    }

    class Node {
        Node(String key, String value) {
            this.key = key;
            this.value = value;
        }

        Node pre;
        Node next;
        String key;
        String value;
    }

    public static void main(String[] args) {
        LRUCache lruCache = new LRUCache(5);
        lruCache.put("001", "test1");
        lruCache.put("002", "test1");
        lruCache.put("003", "test1");
        lruCache.put("004", "test1");
        lruCache.put("005", "test1");
        lruCache.get("002");
        lruCache.put("004", "test2");
        lruCache.put("006", "test6");
        System.out.println(lruCache.get("001"));
        System.out.println(lruCache.get("006"));
    }

}
```

## Redis 相比 memcached 有哪些优势？

1. memcached 所有的值均是简单的字符串，redis 作为其替代者，支持更为丰富的数据类型
2. redis 的速度比 memcached 快很多
3. redis 可以持久化其数据

## Memcache 与 Redis 的区别都有哪些？

1. **存储方式** Memecache 把数据全部存在内存之中，**断电后会挂掉**，**数据不能超过内存大小**。Redis 有部份存在硬盘上，这样能保证数据的持久性。
2. **数据支持类型** Memcache 对数据类型支持相对简单。 Redis 有复杂的数据类型。
3. **使用底层模型不同** 它们之间底层实现方式以及与客户端之间**通信的应用协议**不一样。Redis 直接自己构建了 VM 机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

## Redis 常见性能问题和解决方案

1. `Master`写内存快照，`save`命令调度`rdbSave`函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以`Master`最好不要写内存快照。
2. `Master AOF`持久化，如果不重写`AOF`文件，这个持久化方式对性能的影响是最小的，但是`AOF`文件会不断增大，`AOF`文件过大会影响`Master`重启的恢复速度。`Master`最好不要做任何持久化工作，包括内存快照和`AOF`日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个`Slave`开启`AOF`备份数据，策略为每秒同步一次。
3. `Master`调用`BGREWRITEAOF`重写`AOF`文件，`AOF`在重写的时候会占大量的`CPU`和内存资源，导致服务`load`过高，出现短暂服务暂停现象。
4. Redis 主从复制的性能问题，为了主从复制的速度和连接的稳定性，`Slave`和`Master`最好在同一个局域网内

## 请用 Redis 和任意语言实现一段恶意登录保护的代码，限制 1 小时内每用户 Id 最多只能登录 5 次。具体登录函数或功能用空函数即可，不用详细写出。

> 用列表实现:列表中每个元素代表登陆时间,只要最后的第 5 次登陆时间和现在时间差不超过 1 小时就禁止登陆.用 Python 写的代码如下：

``` bash
#!/usr/bin/env python3

import redis
import sys
import time

r = redis.StrictRedis(host='127.0.0.1', port=6379, db=0)
try:
    id = sys.argv[1]
except:
    print('input argument error')
    sys.exit(0)
if r.llen(id) >= 5 and time.time() – float(r.lindex(id, 4)) <= 3600:
    print("you are forbidden logining")
else:
    print('you are allowed to login')
    r.lpush(id, time.time())
    # login_func()
```

## 为什么 redis 需要把所有数据放到内存中?

Redis 为了达到最快的读写速度将数据都读到内存中，并通过异步的方式将数据写入磁盘。所以 redis 具有快速和数据持久化的特征。

**如果设置了最大使用的内存，则数据已有记录数达到内存限值后不能继续插入新值。**

## Redis 是单进程单线程的?

Redis 利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销

## Redis 的并发竞争问题如何解决?

> Redis 为单进程单线程模式，采用队列模式将并发访问变为串行访问。

Redis 本身没有锁的概念，Redis 对于多个客户端连接并不存在竞争

但是在 Jedis 客户端对 Redis 进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是由于客户端连接混乱造成。对此有 2 种解决方法：

1. **客户端角度**，为保证每个客户端间正常有序与 Redis 进行通信，对**连接进行池化**，同时对客户端读写 Redis 操作采用内部锁`synchronized`。
2. **服务器角度**，利用`setnx`实现锁。

## Redis 事物的了解 CAS(check-and-set 操作实现乐观锁 )?

> 在 Redis 中，`MULTI`/`EXEC`/`DISCARD`/`WATCH`这四个命令是我们实现事务的基石。

**事务的实现特征：**

1. 在事务中的所有命令都将会被**串行化**的顺序执行，事务执行期间，Redis**不会**再为其它客户端的请求提供任何服务，从而保证了事物中的所有命令被原子的执行。
2. 和关系型数据库中的事务相比，**在 Redis 事务中如果有某一条命令执行失败，其后的命令仍然会被继续执行。**
3. 可以通过`MULTI`命令开启一个事务，在该语句之后执行的命令都将被视为事务之内的操作。
4. 在事务开启之前，如果客户端与服务器之间出现**通讯故障**并导致网络断开，**其后所有待执行的语句都将不会被服务器执行**。然而如果网络中断事件是发生在客户端执行`EXEC`命令之后，**那么该事务中的所有命令都会被服务器执行**。
5. 当使用`Append-Only`模式时，Redis 会通过调用系统函数`write`将该事务内的所有写操作在本次调用中全部写入磁盘。然而如果在写入的过程中出现系统崩溃，如电源故障导致的宕机，那么此时也许只有部分数据被写入到磁盘，而另外一部分数据却已经丢失。

## Redis 持久化的几种方式

### 快照（snapshots）

> 缺省情况情况下，Redis 把数据快照存放在磁盘上的二进制文件中，文件名为 dump.rdb。

**工作原理**

- Redis forks.
- 子进程开始将数据写到临时 RDB 文件中。
- 当子进程完成写 RDB 文件，用新文件替换老文件。
- 这种方式可以使 Redis 使用`copy-on-write`技术。

### AOF

快照模式并不十分健壮，当系统停止，或者无意中 Redis 被 kill 掉，最后写入 Redis 的数据就会丢失。

这对某些应用也许不是大问题，但对于要求高可靠性的应用来说，Redis 就不是一个合适的选择。

Append-only 文件模式是另一种选择。

### 虚拟内存方式

当你的 key 很小而 value 很大时,使用 VM 的效果会比较好.因为这样节约的内存比较大。

当你的 key 不小时,可以考虑使用一些非常方法将很大的 key 变成很大的 value,比如你可以考虑将 key,value 组合成一个新的 value。

`vm-max-threads`这个参数,可以设置访问 swap 文件的线程数,设置最好不要超过机器的核数,如果设置为 0,那么所有对 swap 文件的操作都是串行的。

可能会造成比较长时间的延迟,但是对数据完整性有很好的保证。
