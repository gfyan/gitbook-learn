---
description: 这是redis中间件相关知识笔记
---

# redis部分

### 说说redis，它是什么？可以用来做什么？
redis是一个缓存中间件，是一个key-value类型的内存数据库，整个数据都是存在内存中的，所以redis读写性能相对较高，单机qps最高可高达10万次，在内存中的数据会定期通过异步操作把数据库数据flush到硬盘上进行保存。

### redis的数据结构有哪些，你都用过哪些数据，具体原理都了解吗？
redis支持很多类型的数据结构，常见的有string、hash、set、list、sortedSet，HyperLogLog、Geo、Bitmap，BloomFilter、RedisSearch、Redis-ML、JSON，redis5.0添加了Stream（流）的数据结构。

1. string（字符串），redis的字符串并非使用的是c语言底层的字符串，而是自己单独写了一个sds结构的动态字符串，该动态字符串类似于java里面的ArrayList，具有预分配内存、扩容、获取字符串长度为O(1)复杂度，sds内部结构封装了四个变量，总共分配的数组容量、已使用的数组容量、特殊标志位（占用1bytes，低）、数组内容。redis的字符串储存形式分为两种一种紧凑型一种非紧凑型，紧凑型是redisObject和sds连续分配内存，非紧凑RedisObject和sds是分开分配的，需要两次内存分配。
扩容机制：1M以内redis字符串扩容机制是100%扩容，超过1M扩容机制是每次多分配1M内存。<br>
max：最大长度不允许超过512M。<br>
embstr和raw形式：redis字符串超过44bytes会按照raw形式储存，44bytes以内都会以embstr储存，redisObject和sds一起分配内存。注意点：对字符串进行append也会变为raw形式储存，因为redis并未对embstr编码的字符串对象编写任何修改程序，换一句话说embstr是只读的，只有int和raw编码的对象字符串才可以修改。<br>

2. hash（哈希表），hash表示内存采用的是字典结构，字典是redis最常见且用途最广的数据结构，redis字典内部包含两个hashtable，通常情况下只有一个哈希表有值，另外一个在扩容的时候会存在值，字典除了hash会用以外，Zset也会用到，zSet里面的value-score的映射关系就是字典结构实现的。set用的也是字典结构，只不过set所有的value都是null<br>
扩容机制：一般当hash表中的所有元素个数等于一位数组的长度时会进行扩容，但是如果redis再进行bgsave的话，为了减少内存页的分离会尽量不扩容，但是如果hash表变的非常臃肿，元素个数已经达到了一维数组的5倍时，会进行强制扩容。<br>
扩容大小：新数组是原数组大小的2倍。<br>
缩容机制：当hash表中的所有元素低于一维数组长度的10%，redis会考虑对该hash表进行缩容。<br>

3. list（列表），类似于java中的LinkedList，但是redis的列表有所不同，最早期采用的也是双向链表来实现，但是双向链表会产生内存较多的内存碎片问题，所以redis后面对列表进行了改进，改成了快速列表（quicklist），quicklist是ziplist（压缩列表）+linkedlist（双向列表）结合体，它将linkedlist按段进行压缩，每一段用ziplist紧凑储存，多个ziplist使用双向指针串联起来。默认单个ziplist储存大小为8k，超过便会新起一个ziplist，ziplist储存大小由list-max-ziplist-size控制。


4. set（无序集合、不可重复），set集合采用的hashtable结构，只不过value为null，当数据较少的时候set采用的是intset结构。

5. zset（有序集合），zset有序集合，底层采用的是跳跃表来实现，不过zset在元素介绍的时候会采用压缩列表储存，以达到节约内存的目的。<br>
为什么zset采用跳跃表而不使用AVL或者红黑树？原因有四点，1.跳跃表相比AVL和红黑树更适合做范围查找。2.平衡树和红黑树的插入以及删除可能引起子树的调整，操作较为复杂。3.从内存上来说跳跃表比平衡树更灵活一些，平衡树每个节点都会有包含指向左右节点的两个指针，但是跳跃表并不是，按照公式每个节点包含指针为1/(1-p)。

