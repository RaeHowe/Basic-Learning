## MySQL 有什么调优的方式？

* 1.根据不同的业务场景，选择使用合适的存储引擎。例如如果涉及到事务需要保证数据可靠性，一致性的话  
  那么就选择InnoDB存储引擎，如果是读操作远远多于写操作的话，就使用MyISAM存储引擎。
* 2.对InnoDB的buffer pool的大小设置。根据实际服务器的内存大小来决定InnoDB存储引擎的buffer pool  
  的大小，不能过大或者过小。buffer pool的大小可以通过innodb_buffer_pool_size来进行设置。
* 3.数据预热。在默认情况下，针对于某条数据的读取操作之后，才会把该条数据放置到buffer pool里面去，所以  
  可以针对于该情况，在数据库刚刚启动后，可以通过数据预热的方式，将磁盘上的部分访问量大的数据缓存到内存中。
  可以加快数据读取的速度。
* 4.建立合适的索引。通过索引，可以加快数据的访问速度，表与表之间的连接速度，以及group by和sort操作的  
  检索速度。但是针对于索引所带来的一些不良影响，比如占用空间、索引的维护等等，需要进行良好的设计。
* 5.表字段数据类型的选择。一般情况下，选择那些可以满足了业务数据长度大小的数据类型即可，避免使用过大的数据  
  类型来保存，造成空间浪费。