# 项目基本情况和可优化点
最初的是时候，我是基于传统的 BIO 的方式 Socket 进行网络传输，然后利用 JDK 自带的序列化机制 来实现这个 RPC 框架的。后面，我对原始版本进行了优化，已完成的优化点我都列在了下面
 使用 Netty（基于 NIO）替代 BIO 实现网络传输；
 使用开源的序列化机制 Kyro（也可以用其它的）替代 JDK 自带的序列化机制；
 使用 Zookeeper 管理相关服务地址信息
 Netty 重用 Channel 避免重复连接服务端
 增加 Netty 心跳机制 : 保证客户端和服务端的连接不被断掉，避免重连。
 客户端调用远程服务的时候进行负载均衡 ：调用服务的时候，从很多服务地址中根据相应的负载均衡算法选取一个服务地址。ps：目前实现了随机负载均衡算法与一致性哈希算法。
 客户端与服务端通信协议（数据包结构）重新设计 ，可以将原有的 RpcRequest和 RpcReuqest 对象作为消息体，然后增加如下字段（可以参考：《Netty 入门实战小册》和 Dubbo 框架对这块的设计）：
  魔数 ： 通常是 4 个字节。这个魔数主要是为了筛选来到服务端的数据包，有了这个魔数之后，服务端首先取出前面四个字节进行比对，能够在第一时间识别出这个数据包并非是遵循自定义协议的，也就是无效数据包，   为了安全考虑可以直接关闭连接以节省资源。
  序列化器编号 ：标识序列化的方式，比如是使用 Java 自带的序列化，还是 json，kyro 等序列化方式。
  消息体长度 ： 运行时计算出来。
