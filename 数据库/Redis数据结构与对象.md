# 数据结构与对象-SDS、链表、字典

Redis里每个键值对都由对象组成

- 数据库的键总是一个字符串对象
- 数据库键的值为**字符串对象、列表对象、哈希对象、集合对象、有序集合**对象五种

## 简单动态字符串（simple dynamic string ,SDS）

Redis中只有无需修改的字面值常量才会使用c的字符串（例如日志打印），其他情况都使用SDS

> SET msg "hello world"
>
> - 键为字符串对象，为“msg”的SDS
> - 值也是字符串对象，“hello”的SDS

除了用于保存字符串值，SDS还被用做**缓冲区**：如AOF模块的AOF缓冲区，客户端状态中的输入缓冲区

### SDS定义

```cpp
struct sdshdr {
    // buf 中已占用空间的长度
    int len;	
    // buf 中剩余可用空间的长度
    int free;	
    // 数据空间
    char buf[];	
};
```

![示例](https://cdn.konyue.site/image-20220524152548769.png)

### SDS与C字符串的区别

- 由于维护了len，可以$O(1)$获取字符串的长度，而C中需要$O(n)$
- 解决拼接字符串时的缓冲区溢出问题，只有free满足时进行拼接，不满足会进行内存分配扩容
- 减少修改字符串带来的内存重分配次数
    - C中字符串在增长和缩短的时候都需要重新分配内存
    - SDS实现了空间预分配和惰性空间释放的优化策略
        - 空间预分配
            - 修改后小于1M时，len=buff，会分配同等空间
            - 修改后大于1M时，会为buff数组分配1M空间
            - 通过上述策略，SDS连续增长N次字符串内存重分配次数从必定N次降低为最多N次
        - 惰性空间释放
            - 优化字符串缩短操作，当缩短后，并不立刻重新分配内存，而是用free记录，等待再次使用
            - 如果有需要真正释放未使用的空间，SDS提供了API
    - SDS是二进制安全的
        - 所有SDS API都会以处理二进制的方式处理SDS存放在buf数组中的数据，数据写入的时候是什么样，被读取的时候就是什么样，使得Redis可以保存任意格式的二进制数据
    - 兼容部分C字符串函数

## 链表

链表被用于实现Redis的各种功能，比如列表键、发布与订阅、慢查询、监视器等

### 定义

listNode结构

```cpp
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
} listNode;
```

list结构

```cpp
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
    // 链表所包含的节点数量
    unsigned long len;
} list;
```

![](https://cdn.konyue.site/image-20220525000002663.png)



### Redis链表特性：

- 双端
- 无环
- 带表头指针和表尾指针
- 带链表长度计数器
- 多态：使用 void* 保存值，可以通过list结构的dup、free、match三个属性为节点值设置类型特定函数，可以保存各种不同类型的值

## 字典

Redis的字典使用哈希表作为底层实现，一个哈希表里可以有多个哈希表节点，而每个哈希表节点保存了字典中的一个键值对

### 定义

#### 哈希表定义

```cpp
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
```

![](https://cdn.konyue.site/image-20220525001656296.png)

#### 哈希表节点定义

```cpp
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

- key属性保存键值对中的键，v属性保存键值对中的值，其中键值对的值可以是一个指针/uint64_t整数/int64_t整数
- next属性是指向另一个哈希表节点的指针，可以将多个哈希值相同的键值对连接在一起，解决键冲突问题

![](https://cdn.konyue.site/image-20220525002137142.png)

#### 字典定义

```cpp
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */
} dict;
```

type属性和privdate属性是针对不同类型的键值对，为创建多态字典而设置的：

- type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函数
- 而privdata属性则保存了需要传给那些类型特定函数的可选参数

```cpp
typedef struct dictType {
    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);
    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);
    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);
    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

ht属性是一个包含2个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht[0] 哈希表，ht[1] 哈希表只会在对ht[0] 哈希表进行 rehash 时使用

另一个rehash有关的属性是rehashidx, 记录rehash 当前的进度，如果目前没有在进行 rehash ，那么他的值为-1

![没有rehash的字典](https://cdn.konyue.site/image-20220525003255677.png)



### 哈希算法

Redis计算哈希值和索引值的方法：

```cpp
//计算键key的哈希值
hash = dict->type->hashFunction(key);

//使用哈希表的sizemark属性和哈希值，计算索引值
//根据情况，ht[x]可以是ht[0]或者ht[1]
index = hash & dict->ht[x].sizemark;
```

Redis使用MurmurHash2算法来计算哈希值

```cpp
unsigned int murMurHash(const void *key, int len)
{
    const unsigned int m = 0x5bd1e995;
    const int r = 24;
    const int seed = 97;
    unsigned int h = seed ^ len;
    // Mix 4 bytes at a time into the hash
    const unsigned char *data = (const unsigned char *)key;
    while(len >= 4)
    {
        unsigned int k = *(unsigned int *)data;
        k *= m; 
        k ^= k >> r; 
        k *= m; 
        h *= m; 
        h ^= k;
        data += 4;
        len -= 4;
    }
    // Handle the last few bytes of the input array
    switch(len)
    {
        case 3: h ^= data[2] << 16;
        case 2: h ^= data[1] << 8;
        case 1: h ^= data[0];
            h *= m;
    };
    // Do a few final mixes of the hash to ensure the last few
    // bytes are well-incorporated.
    h ^= h >> 13;
    h *= m;
    h ^= h >> 15;
    return h;
}
```

### 解决键冲突

Redis的哈希表使用链地址法解决哈希冲突，每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的节点可以用这个单向链表连接起来，解决了键冲突的问题

因为dictEntry节点组成的链表没有指向链表表尾的指针，程序总是将新节点添加到链表的表头位置。排在已有节点的前面

### rehash

随着操作的执行，哈希表保存的键值对会逐渐增多或者减少，为了人哈希表的负载因子维持在一个合理范围内，哈希表保存的键值对数量太多或者太少，会对哈希表的大小进行扩容或者收缩

#### rehash步骤

- 为字典的 ht[1] 哈希表分配空间
    - 如果是扩展，那么 ht[1]  的大小为第一个大于等于 ht[0].used*2 的 $2^n$
    - 如果是收缩，那么 ht[1]  的大小为第一个大于等于 ht[0].used 的 $2^n$

- 将保存在 ht[0] 中的键值对 rehash 到 ht[1] 上：rehash 指的是重新计算键的哈希值和所有值，然后将键值对放置到 ht[1] 哈希表的指定位置上
- 当 ht[0] 上包含的所有键值对都迁移到了 ht[1] 之后，释放 ht[0], 将 ht[1] 设置为 ht[0] ,并在 ht[1] 新创建一个空白哈希表，为下一次 rehash做准备

#### 负载因子：

`load_factor = ht[0].used / ht[0].size `

- 服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且哈希表的负载因子大于等于1。
- 服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且哈希表的负载因子大于等于5。

需的负载因子并不相同，这是因为在执行BGSAVE命令或BGREWRITEAOF命令的过程中，Redis需要创建当前服务器进程的子进程，而大多数操作系统都采用**写时复制**(copy-on-write)技术来优化子进程的用效率,所以在子进程存在期间，服务器会提高执行扩展操作所需的负载因子，从而尽可能地避免在子进程存在期间进行哈希表扩展操作，这可以**避免不必要的内存写人操作**，最大限度地节约内存。

- 当哈希表的负载因子小于0.1时，程序自动开始对哈希表执行收缩操作

### 渐进式 rehash

rehash动作并不是一次性，集中式的完成的，而是分多次，渐进式的完成的

- 这样做是为了避免一次性大量键值对全部 rehahs带来的计算量太大导致服务器停止服务

渐进式 rehash 的步骤

1. 为 ht[1] 分配空间，让字典同时持有 ht[0] ht[1] 两个哈希表
2. 在字典中维持一个索引计数器变量 rehashidx ，将它的值设置为0.表示 rehash 工作正式开始
3. 在 rehash 进行期间， 每次对字典执行增删查改操作时，字典除了执行指定操作外，还会顺带将 ht[0] 哈希表在 rehashidx 索引上所有键值对 rehash 到 ht[1] ，当 rehash 工作完成后，程序将 rehashidx 属性的值+1
4. 最终在某个时间点，ht[0] 的所有键值对都会被 rehash 至 ht[1] ,这时将 rehashidx 属性的值设为 -1 ,表示 rehash 操作已完成

#### rehash 执行期间的哈希表操作

- 在 rehash 过程中，字典会同时使用 ht[1] 和 ht[0] 两个哈希表，所有**删查改**等操作会在2个哈希表上执行

- 新增的键值对会一律保存到 ht[1] 里面，不会对 ht[0] 进行增加数据的操作

























