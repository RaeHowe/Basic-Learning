## 简述 HTTP 短链接与长链接的区别

* 题外话:
  * HTTP的长连接和短连接本质上是TCP长连接和短连接。HTTP属于应用层协议，在传输层使用TCP协议，  
    在网络层使用IP协议。IP协议主要解决网络路由和寻址问题，TCP协议主要解决如何在IP层之上可靠的  
    传递数据包，使在网络上的另一端收到发端发出的所有包，并且顺序与发出顺序一致。TCP有可靠，面向  
    连接的特点。

* 在HTTP1.0版本中，默认是使用短连接的，也就是说浏览器和服务器每进行一次HTTP操作，就建立一次连接，
  当任务结束就中断连接。在HTTP1.1版本中，默认是长连接，用以保持连接特性。使用长连接的HTTP协议，  
  会在响应头有加入这行代码：Connection:keep-alive。在使用长连接的情况下，当一个网页打开完成后，  
  客户端和服务器之间用于传输HTTP数据的 TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会  
  继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器  
  软件（如Apache）中设定这个时间。实现长连接要客户端和服务端都支持长连接。

* 短连接的优点
  * 管理起来比较简单，存在的连接都是有用的连接，不需要额外的控制手段。
* 长连接的优点
  * 长连接可以省去较多的TCP建立和关闭的操作，减少浪费，节约时间。对于频繁请求资源的客户来说，较适用长连接。
