# Innodb MVCC

> 只要你操作数据库，你就会有高并发场景。这个场景就是数据库的增删改查

> 有并发就会引起线程不安全。([百度-线程安全](<https://baike.baidu.com/item/%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8/9747724?fr=aladdin>))

> 如何管理并发，我们就需要锁机制来控制并发。而MVCC就是一套管理锁的方法

## [百度-MVCC](<https://baike.baidu.com/item/MVCC/6298019?fr=aladdin>)

Multi-Version Concurrency Control 多版本[并发控制](https://baike.baidu.com/item/%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6/3543545)，MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问；在编程语言中实现事务内存。

### MVCC中的锁 [百度-共享锁](<https://baike.baidu.com/item/%E5%85%B1%E4%BA%AB%E9%94%81/11032319>) 和 [百度-排它锁](<https://baike.baidu.com/item/%E6%8E%92%E5%AE%83%E9%94%81/11057879?fr=aladdin>)



> 共享锁（Shared Locks 或 s锁），又称为读锁，可以查看但无法修改和删除的一种数据锁。

> 排它锁（Exclusive Locks 或 X锁)），又称为写锁，若[事务](https://baike.baidu.com/item/%E4%BA%8B%E5%8A%A1/5945882)T对[数据对象](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%AF%B9%E8%B1%A1/3227125)A加上X锁，则只允许T读取和修改A，其它任何事务都不能再对A加任何类型的锁，直到T释放A上的锁。它防止任何其它事务获取资源上的锁，直到在事务的末尾将资源上的原始锁释放为止。

在并发情况下的会出问题的条件

>多线程操作<font color="red">读-读</font>的逻辑，数据获取不会出现问题
>
>多线程操作<font color="red">读-写，写-读</font>的逻辑，不控制好数据，出现数据于逻辑不一致的情况。



简单理解S锁

> 例如 有一个T线程给A数据添加了S锁。那么在多线程情况下，所有线程可以<font color="red">查看</font>A数据，但所有线程不可以<font color="red">修改</font>A数据。所以S锁限制了对数据修改的操作，可以保证数据不会出现<font color="red">脏读</font>的情况。操作例子->SELECT ... LOCK IN SHARE MODE

简单理解X锁

> 例如 有一个T线程给A数据添加了X锁。那么在多线程情况下，其他线程不可以<font color="red">查看</font>A数据，其他线程不可以<font color="red">修改</font>A数据，而只有T线程才可以<font color="red">查看</font>A数据，<font color="red">修改</font>A数据。所以X锁限制了其他线程对数据操作的权利，可以保证数据不会出现<font color="red">脏读</font>的情况。**添加X锁会造成其他线程的阻塞，这是需要注意的问题。**操作例子->SELECT ... FOR UPDATE

S锁升级到X锁

> 例如，有一个T线程给A数据加S锁，有来一个TT线程给A数据加X锁，这个因为T线程是先进入的，所以TT线程不能锁定T线程，但TT线程就插队，把S锁线程都阻塞了。**这样可以提高数据库的修改效率，也有效防止了脏读问题**





## 行锁的3中算法



- Record Lock :单个行记录上的锁

- Gap Lock : 间隙锁，锁定一个范围，但不包含记录本身

- Next-Key :Record Lock+Gap Lock,锁定一个范围，并且锁定记录本身。

  

未完待续。。。。











## 参考

《MySQL技术内幕  InnoDB存储引擎(第2版.姜承尧)》

[掘金-MySQL 是怎样运行的：从根儿上理解 MySQL](https://juejin.im/book/5bffcbc9f265da614b11b731/section)