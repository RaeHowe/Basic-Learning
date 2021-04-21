## Redis 有几种数据结构？Zset 是如何实现的？

* Redis的基本数据结构
  * string
  * hash
  * set
  * zet
  * list

* string
  * redis中的字符串是通过SDS来实现的。SDS的结构大致如图所示:  
    ![avatar](./../PIC/SDS.png)
  free代表了还剩多少空间；len代表了字符串长度；buf代表了具体存放的字符数组内容
