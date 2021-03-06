## TCP 四次挥手的时候，如果有大量的TIME_WAIT和CLOSE_WAIT的话，分别怎么处理？

* 大量TIME_WAIT的处理方法
  * 我们考虑高并发短连接的业务场景，在高并发短连接的 TCP 服务器上，当服务器处理完请求后主动请求关闭  
    连接，这样服务器上会有大量的连接处于 TIME_WAIT 状态，服务器维护每一个连接需要一个 socket，也  
    就是每个连接会占用一个文件描述符，而文件描述符的使用是有上限的，如果持续高并发，会导致一些正常的  
    连接失败。解决方案：修改配置或设置 SO_REUSEADDR 套接字，使得服务器处于 TIME-WAIT 状态下的端  
    口能够快速回收和重用。


* 大量的CLOSE_WAIT怎么解决
  * 首先检查是不是自己的代码问题（看是否服务端程序忘记关闭连接），如果是，则修改代码。调整系统参数，  
    包括句柄相关参数和 TCP/IP 的参数，一般一个 CLOSE_WAIT 会维持至少 2 个小时的时间，我们可以  
    通过调整参数来缩短这个时间。

