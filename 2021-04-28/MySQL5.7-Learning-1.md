## 学习MySQL5.7文档第一天

### 8.0版本中文文档: http://www.deituicms.com/mysql8cn/cn/web.html
### 5.7版本中文文档: https://www.docs4dev.com/docs/zh/mysql/5.7/reference/innodb-storage-engine.html

* 1.InnoDB存储引擎的优势
DML操作遵循ACID模型，其中事务具有commit, rollback和crash-recovery功能来保证数据可靠性。
行级锁和一致性读提高了多用户并发性和性能。
InnoDB的表将数据安排到磁盘上，以基于主键进行查询的优化。每个表都有一个聚簇索引，该索引对数据进行组织，以最大程度减少主键查找的I/O
支持了外键约束，使用外键检查插入、更新和删除操作，以确保这些操作不会导致不同table之间的不一致。

* 2.InnoDB存储引擎的表特性
数据库发生宕机之后，InnoDB的crash recovery机制可以保证恢复重启之前所有已经提交事务的更改；撤销所有未提交事务的更改。
InnoDB存储引擎维护了自己的buffer pool，buffer pool在访问数据的时候，会将磁盘上面表的数据和索引的数据放置到内存中，经常被访问的数据会从内存中直接读取。在专用数据库服务器上，通常将80%的物理内存分配给buffer pool使用。
当从表中一遍又一遍地查询相同的数据行时，称为自适应哈希指数的功能将接管这些查找操作，使其变得更快，就像它们从哈希表中查询出来的一样。
可以压缩表和表相关的索引。

* 3.InnoDB存储引擎中表的最佳用法
使用一个或者多个最被频繁查询的列作为表的主键。如果没有指定主键的话，InnoDB内部会附加一个额外的列作为主键，并且该列是自增的。
在进行多表的join操作的时候，在需要进行连接的列上面定义外键，然后在每个表中使用相同的数据类型来定义需要连接的列。添加外键可以确保对引用的列进行索引，从而可以提高查询性能。外键还会将删除或者更新操作传播到所有受影响的表中，并且如果表中不存在相对应的ID的话，则可以防止在子表中插入数据。
如果关闭了 AUTOCOMMIT了的话，事务提交会受到性能影响。

* 4.InnoDB针对于ACID模型的具体实现
ACID模型是一组数据库设计原则，这些原则保证了数据库的可靠性，不会因为数据库系统的故障异常等问题而使得数据发生失真。
  * A （原子性）
InnoDB对于原子性的保证主要涉及到了事务的概念。相关的MySQL功能包括：
AUTOCOMMIT
COMMIT
ROLLBACK
  * C （一致性）
  InnoDB对于一致性的保证是通过存储引擎内部的机制来保证的：
  Double write机制 （http://blog.chinaunix.net/uid-20785090-id-4364051.html）
  Crash recovery机制
  * I （隔离性）
  InnoDB中的隔离性主要是基于事务所提供的。目前InnoDB所支持的隔离级别包括了：
  Read Uncommited
  Read Commited
  Repeatable Read
  Serializable
  * D （持久型）
  InnoDB的持久型方面主要是通过redo log来进行保证的。

InnoDB的MVCC
InnoDB是多版本存储引擎，它保留有关已经被更改数据行的旧版本信息，以支持诸如并发和回滚的事务功能。数据行的旧版本信息被称为“Rollback Segment”的数据结构，被存储在表空间中。InnoDB使用回滚段中的信息来执行事务回滚中所需的撤销操作，此数据信息还被用来实现“一致性读”的功能。
内部实现上，InnoDB向存储在数据库中的每一行数据都添加了三个字段信息。分别是DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID.
DB_TRX_ID：占用6个字节，用来表示插入或更新该行数据的最后一笔交易的交易ID。此外，删除操作在MVCC整体概念中也被归纳到了更新操作，在该更新操作中，如果本质是一个删除操作的话，那么数据行的头信息（record header）里有一个专门的bit位（delete_flag）用来表示当前记录是否已经被删除。
DB_ROLL_PTR：占用7个字节，被称为“回滚指针”。由于MVCC中所有旧版本的数据记录都被保存在了undo log文件中，并且通过链表的形式来组织，所以该指针指向了当前数据记录项的Rollback segment的undo log记录
DB_ROW_ID：占用6个字节，包含了行的ID信息。如果InnoDB自动生成了聚簇索引（就是说如果表如果存在主键的话），那么该聚簇索引包含行ID信息；如果表没有指定主键的话，那么DB_ROW_ID的值不会出现在任何索引中。
undo log的类型
在回滚段中的undo log文件，大体上可以分为了插入undo log和更新undo log两种：
插入undo log是指在insert操作中产生的undo log。由于insert操作所产生的数据内容，只能在当前事务中可见，其他事务不可见。所以undo log可以在事务提交之后直接删除，而不需要purge操作。
更新undo log是指在delete和update操作中产生的undo log。该undo log会被后续用于MVCC当中，因此不能在事务提交的时候就将其删除。事务提交之后会将该日志记录放置到undo log链表中，等待purge线程在合适的时候进行删除。
purge机制
在InnoDB的MVCC中，当执行删除的SQL语句时，并不会立即将数据从磁盘上面物理删除。而是在数据行的record header里面标记该行数据被删除了。这就导致了undo log和数据页的大小发生膨胀的问题，于是在InnoDB中引入了purge机制来回收。
purge是一个线程，主要任务是将数据库中已经被标记删除的数据删除掉，另外也会批量回收undo log。数据库的数据页很多，要清除被删除的数据，不可能遍历所有数据页。由于所有的变更都有undo log记录，因此，从undo log作为切入点，在清理过期的undo log的同时，也将数据页中被删除的记录一并清除。