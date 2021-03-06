## RestFul 与 RPC 的区别是什么？RestFul 的优点在哪里？

* RPC
  * 调用过程原理是socket+动态代理 
  * 优点
    * 调用简单，清晰，不用像REST一样复杂，就像调用本地方法一样简单
    * 高效低延迟，性能高
    * 自定义网络协议（避免HTTP1.1协议版本，请求头过大的问题产生）
    * 性能消耗低，搞笑的序列化协议可以支持高效的二进制传输
    * 自带负载均衡
  * 缺点
    * 代码耦合性更强一些。（调用方应用对微服务提供的抽象接口存在强依赖关系，因此不论开发、测试、集成环境  
      都需要严格的管理版本依赖，才不会出现服务方与调用方的不一致导致应用无法编译成功等一系列问题）
      
* REST
  * 优点
    * 耦合性低，兼容性好，提高开发效率
    * 不用关心接口实现细节，相对更规范，更标准，更通用
  * 缺点
    * 性能低下，不支持自定义网络传输协议
  
对于选择来说
  * RPC适用于内网服务调用，对外提供服务请走REST
  * IO密集的服务调用用RPC，低频服务用REST
  * 服务调用过于密集与复杂，RPC就比较适用