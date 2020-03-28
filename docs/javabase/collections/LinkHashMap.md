# LinkHashMap

散列表 + 双向链表

HashMap 中 Entry 增加 before 和 after 引用。

特点：

- 保证映射顺序
- 允许null键和null值
- 非同步，线程不安全

## 进阶机制

### 高并发

排序模式

插入顺序，accessOrder=false，默认。

访问顺序，accessOrder=true，近期访问最少到近期访问最多的顺序（适合构建 LRU 缓存）。
