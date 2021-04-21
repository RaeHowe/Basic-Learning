## 简述乐观锁以及悲观锁的区别以及使用场景

* 乐观锁
  * 特点
    * 先进行业务操作，不到万不得已不去拿锁。即"乐观"的认为拿锁多半是会成功的，因此在进行完业务  
      操作需要实际更新数据的最后一步再去拿锁就好。
  * 使用场景
    * 比较适合读取操作比较频繁的场景，如果出现大量的写操作，数据发生冲突的概率就会增大，为了保证  
      数据的一致性，应用层需要不断地重新获取数据，这样会增加大量的查询操作，降低了系统的吞吐量

* 悲观锁
  * 特点
    * 悲观锁的特点是先获取锁，再进行业务操作，即"悲观"地认为获取锁是非常有可能失败的，因此要先  
      确保获取锁成功再进行业务操作。通过所说的"一锁二查三更新"指的就是悲观锁。
  * 使用场景
    * 比较适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样  
      会增加大量的锁开销，降低了系统的吞吐量。