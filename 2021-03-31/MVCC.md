# MySQL的MVCC相关内容

### *1.什么是MVCC*

MVCC（多版本并发控制）是数据库引擎中一种处理读写冲突的方法，该方法的目的在于提高数据库高并发场景下的吞吐能力。  
通过对普通的SELECT操作不加锁，而是通过MVCC来访问指定版本的数据行信息，来避免对数据重复加锁的过程。

### *2.MVCC实现原理*

* 隐藏列
    1. DB_TRX_ID记录最近更新这行数据的事务id，大小为6个字节。
    2. DB_ROLL_PRT表示指向该数据行回滚端的指针，大小为7个字节。该数据行上所有的旧版本数据都被保存在undo log  
       文件中，并且通过链表的形式组织，而DB_ROLL_PRT指针的值正是指向undo log中该行的历史记录链表。
    3. DB_ROW_ID行标识（隐藏单调自增ID），大小为6个字节。如果一个表没有指定主键的话，InnoDB会自动生成一个  
    一个隐藏主键，另外每条记录的头信息（record header）里都有一个专门的bit(delete_flag)来表示当前记录是否  
       已经被删除。
* 如何组织Undo log链表
   1. UPDATE：事务对某个数据行进行更新的时候，会给该条数据行加上排它锁，把该行数据复制到Undo log中，DB_TRX_ID和  
    DB_ROLL_PRT都保持不变，然后更新新数据行的DB_TRX_ID为修改该数据行的事务的ID，并将DB_ROLL_PRT指向刚刚复制  
      到undo log链表中的旧版数据记录，通过遍历链表的方式，可以看到这条记录的变迁方式。
   2. INSERT/DELETE：INSERT会产生一条新的数据记录，这条记录的DB_TRX_ID为创建该条数据记录的事务ID信息；  
    DELETE某条数据记录会被看成一种特殊的UPDATE操作，实质上是软删除，而真正删除操作是在事务commit的时候，DB_TRX_ID  
      会记录下删除这条记录的事务ID。
* ReadView实现一致性读
    1. MVCC只运行在Read Committed和Repeated Read两个事务隔离级别之下，当隔离级别是二者之一的时候，SELECT操作  
    就会用到版本链去undo log里面查询指定版本的数据记录。
    2. Repeated Read（可重复读）隔离级别下的ReadView，执行第一个SELECT语句的时候，后续所有的SELECT操作都是复用  
       这个ReadView，将当前系统中的所有活跃事务复制到一个列表生成ReadView（注意是执行第一次SELECT语句生成ReadView）

(相关学习链接：https://blog.csdn.net/linshenyuan1213/article/details/81948964)