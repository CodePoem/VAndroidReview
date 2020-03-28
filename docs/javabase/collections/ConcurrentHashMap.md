# ConcurrentHashMap

Java8之前：

数组 + 散列表

Segment数组 + HashEntry数组 + HashEntry链表

Java8：

散列表：数组 + 链表

Node数组 + Node链表

特点：

- 不保证映射顺序
- 不允许null键和null值
- 线程安全

## 进阶机制

### 高并发

Java8之前：

分离锁，Segment

Java8：

CAS + synchronized
