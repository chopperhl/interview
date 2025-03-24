### 1. HashMap 原理 
1. 数组加链表
2. 初始数组大小是8
3. 容量超过数组大小的0.75时扩容
4. 节点深度超过8时 转变为红黑树
5. index 计算方法 index = hash(key.hashCode()) & len
6. hash方法 h xor h >> 16

### 2 .ConcurrentHashMap

线程安全的HashMap

循环 casTabAt


### 3. Synchronized 关键字

如果对象锁 类锁
