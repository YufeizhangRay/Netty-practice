# Netty-practice  
  
- [BIO/NIO/AIO基础](#bionioaio基础)  
  - [阻塞I/O](#阻塞io)  
  - [非阻塞I/O](#非阻塞io)  
  - [I/O复用](#io复用)  
  - [信号驱动的I/O](#信号驱动的io)  
  - [异步I/O](#异步io)  
- [Java I/O模型](#java-io模型)  
  - [同步阻塞IO](#同步阻塞io)  
  - [非阻塞式IO模型(NIO)](#非阻塞式io模型nio)  
- [Java NIO核心组件](#java-nio核心组件)  
  - [Channel](#channel)  
  - [Buffer](#buffer)  
  - [Selector](#selector)  
  - [unsafe](#unsafe)  
- [Reactor模型](#reactor模型)    
- [什么是Netty](#什么是netty)  
  - [应用领域](#应用领域)
- [Netty模型](#netty模型)  
- [Netty基础概念](#netty基础概念)  
  - [Netty核⼼组件](#netty核组件)  
  - [Channel](#channel)  
  - [EventLoopGroup](#eventloopgroup)  
  - [ChannelPipeline](#channelpipeline)  
- [Netty示例分析](#netty示例分析)  
  - [服务端](#服务端)  
  - [客户端](#客户端)  
- [Netty线程模型](#netty线程模型)  
- [源码分析之ChannelHandler](#源码分析之channelhandler)  
  - [ChannelHandler](#channelhandler)  
  - [ChannelInboundHandler](#channelinboundhandler)  
  - [ChannelOutboundHandler](#channeloutboundhandler)  
  - [ChannelHandlerContext](#channelhandlercontext)  
  - [ChannelPipeline](#channelpipeline)  
- [源码分析之客户端启动](#源码分析之客户端启动)  
  - [创建Channel](#创建channel)  
  - [注册Channel到Selector](#注册channel到selector)  
  - [Connect](#connect)  
- [源码分析之服务端启动](#源码分析之服务端启动)  
  - [bind核⼼流程](#bind核流程)  
  - [注册Channel到EventLoopGroup](#注册channel到eventloopgroup)  
- [源码分析之EventLoop](#源码分析之eventloop)  
  - [关于Reactor的线程模型](#关于reactor的线程模型)  
  - [NioEventLoopGroup与Reactor线程模型的对应](#nioeventloopgroup与reactor线程模型的对应)  
  - [NioEventLoopGroup](#nioeventloopgroup)    
  - [NioEventLoop](#nioeventloop)  
  - [EventLoop与Channel的关联](#eventloop与channel的关联)  
  - [EventLoop的启动](#eventloop的启动)  
- [源码分析之IO处理循环](#源码分析之io处理循环)  
  - [thread的run循环](#thread的run循环)  
  - [IO事件的轮询](#io事件的轮询)  
  - [IO事件的处理](#io事件的处理)  
- [源码分析之任务队列机制](#源码分析之任务队列机制)  
  - [Task的添加](#task的添加)  
  - [任务的执行](#任务的执行)  
  - [整体回顾](#整体回顾)  
  
## I/O模型分析 Netty学习实践 源码分析  
  
### BIO/NIO/AIO基础  
  
#### 阻塞I/O  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E9%98%BB%E5%A1%9EIO.jpeg)  
  
#### 非阻塞I/O  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/非阻塞IO.jpeg)  
  
#### I/O复用  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/IO%E5%A4%8D%E7%94%A8.jpeg)  
  
#### 信号驱动的I/O  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E4%BF%A1%E5%8F%B7%E9%A9%B1%E5%8A%A8IO.jpeg)  
  
#### 异步I/O  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%BC%82%E6%AD%A5IO.jpeg)  
  
### Java I/O模型  
  
#### 同步阻塞IO  
1:1同步阻塞IO通信模型   
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%90%8C%E6%AD%A5%E9%98%BB%E5%A1%9E%E6%A8%A1%E5%9E%8B.jpeg)  
  
M:N形式的同步阻塞IO通信模型 
![](https://github.com/YufeizhangRay/image/blob/master/Netty/MN%E5%90%8C%E6%AD%A5%E9%98%BB%E5%A1%9EIO.jpeg)  
  
#### 非阻塞式IO模型(NIO)  
NIO+单线程Reactor模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%8D%95%E7%BA%BF%E7%A8%8BReactor.jpeg)  
  
NIO+多线程Reactor模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%A4%9A%E7%BA%BF%E7%A8%8BReactor.jpeg)  
  
NIO+主从多线程Reactor模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E4%B8%BB%E4%BB%8EReactor.jpeg)  
  
#### 总结  
阻塞IO和非阻塞IO的区别就在于：应用程序的调用(等待数据准备阶段)是否立即返回！  
同步IO和异步IO的区别就在于：数据访问(等待数据复制阶段)的时候进程是否阻塞！  
  
再往深了说，可以去分析一下操作系统原理。  
我们的应用在运行的时候，操作系统的内存是被分为两个区域的，一个是用户区，另一个是内核区。其中内核区是不允许轻易被应用进程所访问的。  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84.jpeg)  
  
这样做的好处就是可以防止任意的外部应用程序直接访问内核态资源(机器的硬件等)，提高了安全性。  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8.jpeg)  
  
当一个IO发起时，我们的应用作为一个用户进程需要获取到计算机内核存储资源，但是又无法直接访问，于是可能会被挂起。此时会有一个内核区的内核进程去将硬件存储中的数据先拷贝到自己的进程空间中，然后再将数据传递给用户区中的应用进程。   
  
上述过程中，分为两个阶段  
>1.内核进程将硬件存储中的数据拷贝到自己的进程空间(等待数据准备阶段)  
2.将数据传递给用户区中的应用进程(等待数据复制阶段)  
  
在等待数据准备阶段应用程序能直接返回的(没有被挂起)，就是非阻塞，否则(被挂起)就是阻塞。  
在等待数据复制阶段，若是应用程序自己负责去拿去数据，就是同步。若是内核进程负责推送数据，应用程序可以直接使用，就是异步。  
  
### Java NIO核心组件  
  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/NIO%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6.jpeg)  
  
#### Channel  
Channel 也就是通道，底层封装了socket，可以和Buffer相连进行成块的数据传输。后面有详细介绍。  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Buffer.jpeg)  
  
>FileChannel  
DatagramChannel  
SocketChannel  
ServerSocketChannel  
  
#### Buffer  
buffer就是缓冲区，是一个数组，中有三个重要的属性。  
>capacity 总共的大小  
position  现在数据写入的位置  
limit 数据读取的限制位置  
  
刚开辟的buffer  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%BC%80%E8%BE%9Fbuffer.jpeg)  
  
put()方法写入数据   
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%86%99%E5%85%A5buffer.jpeg)  
  
flip()方法，limit限制读取位置  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E8%AF%BB%E5%8F%96buffer.jpeg)  
  
rewind()方法，可以读取未写区域  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E9%87%8D%E8%AF%BBbuffer.jpeg)  
  
读写模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E8%AF%BB%E5%86%99%E6%A8%A1%E5%9E%8B.jpeg)  
  
#### Selector  
一个单线程的轮询选择器，感知已经准备就绪的事件。当客户端注册的感兴趣时间发生时，通知客户端进行操作，之后继续进行轮询检测。  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/selector.jpeg)  
  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8.jpeg)  
  
#### unsafe  
NIO很多时候都不是通过JVM进行操作，大部分情况下会使用堆外内存(直接内存)。而Java是不可以直接操作这些资源，于是JDK提供了unsafe这个类去帮助我们操作底层的硬件。  
  
### Reactor模型  
  
Reactor模型组件     
>Reactor:Reactor是IO事件的派发者(Selector)。  
Acceptor:Acceptor接受client连接，建立对应client的Handler，并向Reactor注册此Handler。  
Handler:和一个client通讯的实体，按这样的过程实现业务的处理。  
  
### 什么是Netty  
  
Netty 是一种可以轻松快速的开发协议服务器和客户端网络应用程序的 NIO 框架，它大大简化了 TCP 或者 UDP 服务器的网络编程，但是你仍然可以访问和使用底层的 API，Netty 只是对其进行了高层的抽象。Netty 的简易和快速开发并不意味着由它开发的程序将失去可维护性或者存在性能问题。Netty 是被精心设计的，它的设计参考了许多协议的实现，比如 FTP，SMTP，HTTP 和各种二进制和基于文本的传统协议，因此 Netty 成功的实现了兼顾快速开发，性能，稳定性，灵活性为一体，不需要为了考虑一方面原因而妥协其他方面。  
  
总结  
>1.异步事件驱动框架，用于快速开发高性能服务端和客户端  
2.封装了JDK底层BIO和NIO模型，提供高度可用的API  
3.自带编码解码器解决拆包粘包问题，用户只用关心业务逻辑  
4.精心设计的Reactor线程模型支持高并发海量连接  
5.自带协议栈，无需用户关心  
  
#### 应用领域  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%BA%94%E7%94%A8%E9%A2%86%E5%9F%9F.jpeg)  
  
>1. 互联网领域:构建高性能RPC框架基础通信组件，阿里分布式服务框架 Dubbo 的 RPC 框架使用 Dubbo 协议进行节点间通信，Dubbo 协议默认使用 Netty 作为基础通信组件，用于实现各进程节点之间的内部通信。  
>2. ⼤数据领域:经典的 Hadoop 的⾼性能通信和序列化组件 Avro 的 RPC 框架，默认采用 Netty 进⾏跨节点通信，它的 Netty Service 基于 Netty 框架二次封装实现。  
>3. 游戏行业:⽆论是手游服务端、还是大型的⽹络游戏，Java 语⾔得到了越来越⼴泛的应⽤。 Netty 作为高性能的基础通信组件，它本身提供了 TCP/UDP 和 HTTP 协议栈，非常⽅便定制和开发私有协议栈，账号登陆服务器、地图服务器之间可以方便的通过 Netty 进行⾼性能的通信。  
>4. 通信行业:Netty 的异步⾼性能、高可靠性和高成熟度的优点，使它在通信行业得到了⼤量的应用。  
  
### Netty模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Netty%E6%A8%A1%E5%9E%8B.jpeg)  
  
### Netty基础概念  
  
#### Netty架构图  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Netty%E6%9E%B6%E6%9E%84%E5%9B%BE.jpeg)  
  
>Core:核心部分，是底层的⽹络通⽤抽象和部分实现。  
Extensible Event Model :可拓展的事件模型。Netty 是基于事件模型的网络应用框架。  
Universal Communication API :通⽤的通信 API 层。Netty 定义了⼀套抽象的通用通信层的 API 。  
Zero-Copy-Capable Rich Byte Buffer :支持零拷⻉特性的 Byte Buffer 实现。  
Transport Services:传输( 通信 )服务，具体的⽹络传输的定义与实现。  
Socket & Datagram :TCP 和 UDP 的传输实现。   
HTTP Tunnel :HTTP 通道的传输实现。  
In-VM Piple :JVM 内部的传输实现。  
Protocol Support :协议支持。Netty 对于⼀些通用协议的编解码实现。例如:HTTP、 Redis、DNS 等等。  
   
#### Netty核⼼组件  
Netty 有如下⼏个核心组件:  
>Bootstrap & ServerBootstrap  
Channel  
ChannelFuture  
EventLoop & EventLoopGroup  
ChannelHandler  
ChannelPipeline  
  
#### Channel   
也就是通道，这个概念是在 JDK NIO 类库里面提供的一个概念，JDK 中其实现类有客户端套接字通道 java.nio.channels.SocketChannel 和服务端监听套接字通道 java.nio.channels.ServerSocketChannel，Channel 的出现是为了支持异步 IO 操作，JDK 里面的通道是 java.nio.channels.Channel。io.netty.channel.Channel 是 Netty 框架自己定义的一个通道接口，Netty 实现的客户端 NIO 套接字通道是 NioSocketChannel，提供的服务器端 NIO 套接字通道是 NioServerSocketChannel。  
  
NioSocketChannel  
客户端套接字通道，内部管理了一个 Java NIO 中的 java.nio.channels.SocketChannel 实例，用来创建 SocketChannel 实例和设置该实例的属性，并调用 Connect 方法向服务端发起 TCP 链接等。 
  
NioServerSocketChannel  
服务器端监听套接字通道，内部管理了一个 Java NIO 中的 java.nio.channels.ServerSocketChannel 实例，用来创建 ServerSocketChannel 实例和设置该实例属性，并调用该实例的 bind 方法在指定端口监听客户端的链接。  
  
Channel与socket的关系  
在 Netty 中 Channel 有两种，对应客户端套接字通道 NioSocketChannel，内部管理 java.nio.channels.SocketChannel 套接字，对应服务器端 监听套接字通道 NioServerSocketChannel，其内部管理自己的 java.nio.channels.ServerSocketChannel 套接字。也就是 Channel 是对 socket 的装饰或者门面，其封装了对 socket 的原子操作。
  
#### EventLoopGroup  
Netty 之所以能提供高性能网络通讯，其中一个原因是因为它使用 Reactor 线程模型。在 netty 中每个 EventLoopGroup 本身是一个线程池，其中包含了自定义个数的 NioEventLoop，每个 NioEventLoop 是一个线程，并且每个 NioEventLoop 里面持有自己的 selector 选择器。在Netty 中客户端持有一个 EventLoopGroup 用来处理网络 IO 操作，在服务器端持有两个 EventLoopGroup，其中 boss 组是专门用来接收客户端发来的 TCP 链接请求的，worker 组是专门用来具体处理完成三次握手的链接套接字的网络 IO 请求的。  
  
Channel与EventLoop的关系  
Netty 中 NioEventLoop 是 EventLoop 的一个实现，每个 NioEventLoop 中会管理自己的一个 selector 选择器和监控选择器就绪事件的线程；每个 Channel 只会关联一个 NioEventLoop；  
当 Channel 是客户端通道 NioSocketChannel 时候，会注册 NioSocketChannel 管理的 SocketChannel 实例到自己关联的 NioEventLoop 的 selector 选择器上，然后 NioEventLoop 对应的线程会通过 select 命令监控感兴趣的网络读写事件。当Channel是服务端通道 NioServerSocketChannel 时候，NioServerSocketChannel 本身会被注册到 boss EventLoopGroup 里面的某一个 NioEventLoop 管理的 selector 选择器上，而完成三次握手 的链接套接字是被注册到了 worker EventLoopGroup 里面的某一个 NioEventLoop 管理的 selector 选择器上；需要注意是多个 Channel 可以注册到同一个 NioEventLoop 管理的 selector 选择器上，这时候 NioEventLoop 对应的单个线程就可以处理多个 Channel 的就绪事件；但是每个 Channel 只能注册到一个固定的 NioEventLoop 管理的 selector 选择器上。  
  
>一个 EventLoopGroup 包含⼀个或多个 EventLoop ，即 EventLoopGroup : EventLoop = 1 : n。  
一个 EventLoop 在它的生命周期内，只能与⼀个 Thread 绑定，即 EventLoop : Thread = 1 : 1。  
所有 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理，从而保证线程安全，即 Thread : EventLoop = 1 : 1 。  
一个 Channel 在它的⽣命周期内只能注册到一个 EventLoop 上，即 Channel : EventLoop = n :1。  
一个 EventLoop 可被分配⾄一个或多个 Channel ，即 EventLoop : Channel = 1 : n 。  
  
当一个连接到达时，Netty 就会创建一个 Channel，然后从 EventLoopGroup 中分配⼀个 EventLoop 来给这个 Channel 绑定上，在该 Channel 的整个生命周期中都是有这个绑定的 EventLoop 来服务的。  
  
#### ChannelPipeline
Netty 中的 ChannelPipeline 类似于 Tomcat 容器中的 Filter 链，属于设计模式中的责任链模式，其中链上的每个节点 就是一个 ChannelHandler。在 netty 中每个 Channel 有属 于自己的 ChannelPipeline，对从 Channel 中读取或者要写 入 Channel 中的数据进行依次处理。  
需要注意一点是虽然每个 Channel(更底层说是每个 socket)有自己的 ChannelPipeline，但是每个 ChannelPipeline 里面可以复用一个 ChannelHandler。   
### Netty示例分析   
   
看此示例前一定要熟悉Java本身的NIO编程，可以参考本仓库上的源代码。  
```
结构:  
├── client   
├── Client.java -- 客户端启动类 ├── ClientHandler.java -- 客户端逻辑处理理类 ├── ClientHandler.java -- 客户端初始化类 

├── server 
├── Server.java -- 服务端启动类 ├── ServerHandler.java -- 服务端逻辑处理理类 ├── ServerInitializer.java -- 服务端初始化类
```
#### 服务端  
首先是编写服务端的启动类。 
```
1  public final class Server {
2    public static void main(String[] args) throws Exception { //Configure the server  
3    //创建两个EventLoopGroup对象
4    //创建boss线程组用于服务端接受客户端的连接
5    EventLoopGroup bossGroup = new NioEventLoopGroup(1);
6    // 创建 worker 线程组⽤于进⾏ SocketChannel 的数据读写 
7    EventLoopGroup workerGroup = new NioEventLoopGroup();
8    try {
9      // 创建 ServerBootstrap 对象
10     ServerBootstrap b = new ServerBootstrap(); 
11     //设置使用的EventLoopGroup
12     b.group(bossGroup,workerGroup)
13     //设置要被实例化的为 NioServerSocketChannel 类 
14     .channel(NioServerSocketChannel.class)
15     // 设置 NioServerSocketChannel 的处理器
16     .handler(new LoggingHandler(LogLevel.INFO))
17     // 设置连入服务端的 Client 的 SocketChannel 的处理器 
18     .childHandler(new ServerInitializer());
19     // 绑定端⼝，并同步等待成功，即启动服务端
20     ChannelFuture f = b.bind(8888);
21     // 监听服务端关闭，并阻塞等待 
22     f.channel().closeFuture().sync();
23     } finally {
24     // 优雅关闭两个 EventLoopGroup 对象 
25     bossGroup.shutdownGracefully(); 
26     workerGroup.shutdownGracefully();
27     }
28   }
29 }
```
>第5到7⾏: 创建两个EventLoopGroup对象。
>>1.boss 线程组: ⽤于服务端接受客户端的连接。  
2.worker 线程组: 用于进行客户端的SocketChannel的数据读写。  
  
>第10⾏: 创建 ServerBootstrap 对象，⽤于设置服务端的启动配置。  
第11⾏: 调⽤ #group(EventLoopGroup parentGroup, EventLoopGroup childGroup) 方法，设置使用的 EventLoopGroup 。  
第14行: 调⽤ #channel(Class<? extends C> channelClass) ⽅法，设置要被实例化的 Channel 为 NioServerSocketChannel 类。在下⽂中，我们会看到该 Channel 内嵌了java.nio.channels.ServerSocketChannel 对象。  
第16行: 调用 #handler(ChannelHandler handler) 方法，设置 NioServerSocketChannel 的处理器。在本示例中，使用了
io.netty.handler.logging.LoggingHandler 类，用于打印服务端的每个事件。  
第18⾏: 调用 #childHandler(ChannelHandler handler) ⽅法，设置连⼊服务端的 Client 的 SocketChannel 的处理器。在本示例中，使用 ServerInitializer() 来初始化连入服务端的 Client 的 SocketChannel 的处理器。  
第20⾏: 先调用 #bind(int port)方法，绑定端⼝，后调用 ChannelFuture#sync() ⽅法，阻塞等待成功。这个过程，就是“ 启动服务端 ”。   
第22⾏: 先调用 #closeFuture() ⽅法，监听服务器关闭， 后调⽤ ChannelFuture#sync() 方法，阻塞等待成功。  
注意，此处不是关闭服务器，而是“ 监听 ”关闭。  
第25到26⾏: 执行到此处，说明服务端已经关闭，所以调⽤ EventLoopGroup#shutdownGracefully() ⽅法，分别关闭两个 EventLoopGroup 对象。  
  
服务端主类编写完毕之后，我们再来设置下相应的过滤条件。这里需要继承Netty中 ChannelInitializer 类，然后重写 initChannel 该方法，进⾏添加相应的设置，例如传输协议设置，以及相应的业务实现类。

```
public class ServerInitializer extends ChannelInitializer<SocketChannel> {
  private static final StringDecoder DECODER = new StringDecoder();
  private static final StringEncoder ENCODER = new StringEncoder();
  private static final ServerHandler SERVER_HANDLER = new ServerHandler();
  @Override
  public void initChannel(SocketChannel ch) throws Exception { 
    ChannelPipeline pipeline = ch.pipeline();
    // 添加帧限定符来防⽌止粘包现象
    pipeline.addLast(new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));
    // 解码和编码，应和客户端一致 
    pipeline.addLast(DECODER); 
    pipeline.addLast(ENCODER);
    // 业务逻辑实现类 
    pipeline.addLast(SERVER_HANDLER); 
  }
}
```
服务相关的设置的代码写完之后，我们再来编写主要的业务代码。使⽤Netty编写业务层的代码，我们需要继承 ChannelInboundHandlerAdapter 或 SimpleChannelInboundHandler 类，在这里顺便说下它们两的区别吧。继承 SimpleChannelInboundHandler 类之后，会在接收到数据后自动 release 掉数据占⽤的 Bytebuffer 资源。并且继承该类需要指定数据格式。而继承ChannelInboundHandlerAdapter 则不会⾃动释放，需要⼿动调用 ReferenceCountUtil.release() 等方法进行释放。继承该类不需要指定数据格式。所以在这⾥，个人推荐服务端继承 ChannelInboundHandlerAdapter ，手动进⾏释放，防止数据未处理完就自动释放了。而且服务端可能有多个客户端进行连接，并且每一个客户端请求的数据格式都不一致，这时便可以进⾏相应的处理。客户端根据情况可以继承 SimpleChannelInboundHandler 类。好处是直接指定好传输的数据格式，就不需要再进行格式的转换了。  
```
@Sharable
public class ServerHandler extends SimpleChannelInboundHandler<String>{
  /**
  - 建⽴连接时，发送⼀条庆祝消息 */
  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    // 为新连接发送庆祝
    ctx.write("Welcome to " + InetAddress.getLocalHost().getHostName() + "!/r/n");
    ctx.write("It is " + new Date() + " now./r/n");
    ctx.flush();
  }
  //业务逻辑处理
  @Override
  public void channelRead0(ChannelHandlerContext ctx, String request) throws Exception  {
    // Generate and write a response.
    String response;
    boolean close = false;
    if (request.isEmpty()) {
      response = "Please type something./r/n";
    } else if ("bye".equals(request.toLowerCase())) {
      response = "Have a good day!/r/n";
      close = true;
    } else {
      response = "Did you say '" + request + "'?/r/n";
    }
      ChannelFuture future = ctx.write(response);
    if (close) {
      future.addListener(ChannelFutureListener.CLOSE);
    }
  }
  @Override
  public void channelReadComplete(ChannelHandlerContext ctx) {
      ctx.flush();
  }
  //异常处理
  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
    cause.printStackTrace();
    ctx.close();
  }
}
```
#### 客户端  
```
public static void main(String[] args) throws Exception {
  EventLoopGroup group = new NioEventLoopGroup();
  try {
    Bootstrap b = new Bootstrap();
    b.group(group)
    .channel(NioSocketChannel.class)
    .handler(new ClientInitializer());
    Channel ch = b.connect("127.0.0.1",8888).sync().channel();
    ChannelFuture lastWriteFuture = null;
    BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
    for (;;) {
      String line = in.readLine();
      if (line == null) {
        break; 
      }
      // Sends the received line to the server.
      lastWriteFuture = ch.writeAndFlush(line + "/r/n");
      // If user typed the 'bye' command, wait until the server closes the connection.
      if ("bye".equals(line.toLowerCase())) {
        ch.closeFuture().sync();
        break;
      } 
    }
    // Wait until all messages are flushed before closing the channel.
    if (lastWriteFuture != null) {
      lastWriteFuture.sync();
    }
  } finally {
    group.shutdownGracefully();
  }
}
```
客户端过滤其这块基本和服务端⼀致。不过需要注意的是，传输协议、编码和解码应该一致
```
public class ClientInitializer extends ChannelInitializer<SocketChannel> {
  private static final StringDecoder DECODER = new StringDecoder();
  private static final StringEncoder ENCODER = new StringEncoder();
  private static final ClientHandler CLIENT_HANDLER = new ClientHandler();
  @Override
  public void initChannel(SocketChannel ch) {
    ChannelPipeline pipeline = ch.pipeline();
    pipeline.addLast(new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));
    pipeline.addLast(DECODER);
    pipeline.addLast(ENCODER);
    pipeline.addLast(CLIENT_HANDLER);
  }
}
```
客户端的业务代码逻辑，主要是打印读取到的信息。  
这里有个注解@Sharable，该注解主要是为了多个handler可以被多个channel安全地共享，也就是保证线程安全。  
```
@Sharable
public class ClientHandler extends SimpleChannelInboundHandler<String> { //打印读取到的数据
  @Override
  protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
    System.err.println(msg);
  }
  //异常数据捕获
  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
  {
    cause.printStackTrace();
    ctx.close()
  }
}
```
### Netty线程模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Netty%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpeg)  
  
### 源码分析之ChannelHandler  
  
ChannelHandler是netty中的核心处理部分，我们使用netty的绝大部分代码都写在这部分，所以了解它的一些机制和特性是很有必要的。  
  
Channel  
Channel接口抽象了底层socket的一些状态属性以及调用方法，针对不同类型的socket提供不同的子类实现。  
   
Channel生命周期  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/channel%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpeg)  
  
#### ChannelHandler  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/handler.jpeg)  
  
ChannelHandler用于处理Channel对应的事件 ChannelHandler接口里面只定义了三个生命周期方法，我们主要实现它的子接口ChannelInboundHandler和ChannelOutboundHandler。为了便利，框架提供了 ChannelInboundHandlerAdapter，ChannelOutboundHandlerAdapter和ChannelDuplexHandler这三个适配类，在使用的时候只需要实现你关注的方法即可。  
  
ChannelHandler生命周期方法  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/ChannelHandler%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%96%B9%E6%B3%95.jpeg)  
  
ChannelHandler里面定义三个生命周期方法，分别会在当前ChannelHander加入ChannelHandlerContext中，从 ChannelHandlerContext中移除，以及ChannelHandler回调方法出现异常时被回调。    
  
#### ChannelInboundHandler
![](https://github.com/YufeizhangRay/image/blob/master/Netty/ChannelInboundHandler.jpeg)  
  
介绍一下这些回调方法被触发的时机
![](https://github.com/YufeizhangRay/image/blob/master/Netty/In%E8%A7%A6%E5%8F%91%E6%97%B6%E6%9C%BA.jpeg)  
  
可以注意到每个方法都带了ChannelHandlerContext作为参数，具体作用是，在每个回调事件里面，处理完成之后，使用ChannelHandlerContext的fireChannelXXX方法来传递给下个ChannelHandler，netty的codec模块和业务处理代码分离就用到了这个链路处理。  
  
#### ChannelOutboundHandler  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/ChannelOutboundHandler.jpeg)  
  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Out%E8%A7%A6%E5%8F%91%E6%97%B6%E6%9C%BA.jpeg)  
  
注意到一些回调方法有ChannelPromise这个参数，我们可以调用它的addListener注册监听，当回调方法所对应的操作完成后，会触发这个监听下面这个代码，会在写操作完成后触发，完成操作包括成功和失败。  
```
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    ctx.write(msg,promise);
    System.out.println("out write");
    promise.addListener(new GenericFutureListener<Future<? super Void>>() {
        @Override
        public void operationComplete(Future<? super Void> future) throws Exception {
            if(future.isSuccess()){
                System.out.println("OK");
            } 
        }
    }); 
}
```
ChannelInboundHandler和ChannelOutboundHandler的区别  
  
区别主要在于ChannelInboundHandler的channelRead和channelReadComplete回调和 ChannelOutboundHandler的write和flush回调上，ChannelOutboundHandler的channelRead回调负责执行入栈数据的decode逻辑，ChannelOutboundHandler的write负责执行出站数据的encode工作。其他回调方法和具 体触发逻辑有关，和in与out无关。  
  
#### ChannelHandlerContext  
每个ChannelHandler通过add方法加入到ChannelPipeline中去的时候，会创建一个对应的 ChannelHandlerContext，并且绑定，ChannelPipeline实际维护的是ChannelHandlerContext 的关系。在 DefaultChannelPipeline源码中可以看到会保存第一个ChannelHandlerContext以及最后一个ChannelHandlerContext的引用。  
```
final AbstractChannelHandlerContext head;
final AbstractChannelHandlerContext tail;
```
而在AbstractChannelHandlerContext源码中可以看到
```
volatile AbstractChannelHandlerContext next;
volatile AbstractChannelHandlerContext prev;
```
每个ChannelHandlerContext之间形成双向链表。  
#### ChannelPipeline  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/ChannelPipeline.jpeg)  
  
在Channel创建的时候，会同时创建ChannelPipeline
```
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```
在ChannelPipeline中也会持有Channel的引用
```
protected DefaultChannelPipeline newChannelPipeline() {
    return new DefaultChannelPipeline(this);
}
```
ChannelPipeline会维护一个ChannelHandlerContext的双向链表
```
final AbstractChannelHandlerContext head;
final AbstractChannelHandlerContext tail;
```
链表的头尾有默认实现
```
protected DefaultChannelPipeline(Channel channel) {
    this.channel = ObjectUtil.checkNotNull(channel, "channel");
    succeededFuture = new SucceededChannelFuture(channel, null);
    voidPromise =  new VoidChannelPromise(channel, true);
    tail = new TailContext(this);
    head = new HeadContext(this);
    head.next = tail;
    tail.prev = head;
}
```
我们添加的自定义ChannelHandler会插入到head和tail之间，如果是ChannelInboundHandler的回调，根据插入的顺序从左向右进行链式调用，ChannelOutboundHandler则相反。  
  
具体关系如下，但是下图没有把默认的head和tail画出来，这两个ChannelHandler做的工作相当重要。
![](https://github.com/YufeizhangRay/image/blob/master/Netty/pipeline.jpeg)  
  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E7%AE%A1%E9%81%93.jpeg)  
  
上面的整条链式的调用是通过Channel接口的方法直接触发的，如果使用ChannelContextHandler的接口方法间接触发，链路会从ChannelContextHandler对应的ChannelHandler开始，而不是从头或尾开始。  

HeadContext  
HeadContext实现了ChannelOutboundHandler，ChannelInboundHandler这两个接口。  
```
class HeadContext extends AbstractChannelHandlerContext implements ChannelOutboundHandler, ChannelInboundHandler
```
因为在头部，所以说HeadContext中关于in和out的回调方法都会触发 关于ChannelInboundHandler， HeadContext的作用是进行一些前置操作，以及把事件传递到下一个ChannelHandlerContext的 ChannelInboundHandler中去看下其中channelRegistered的实现。
```
 public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    invokeHandlerAddedIfNeeded();
    ctx.fireChannelRegistered();
}
```
从语义上可以看出来在把这个事件传递给下一个ChannelHandler之前会回调ChannelHandler的handlerAdded方法而有关ChannelOutboundHandler接口的实现，会在链路的最后执行，看下write方法的实现。
```
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    unsafe.write(msg, promise);
}
```
这边的unsafe接口封装了底层Channel的调用，之所以取名为unsafe，是不需要用户手动去调用这些方法。这个和阻塞原语的unsafe不是同一个也就是说，当我们通过Channel接口执行write之后，会执行 ChannelOutboundHandler链式调用，在链尾的HeadContext ，在通过unsafe回到对应Channel做相关调用。
```
public ChannelFuture write(Object msg) {
    return pipeline.write(msg);
}
```
TailContext  
TailContext实现了ChannelInboundHandler接口，会在ChannelInboundHandler调用链最后执行，只要是对调用链完成处理的情况进行处理，看下channelRead实现。
```
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    onUnhandledInboundMessage(msg);
}
```
如果我们自定义的最后一个ChannelInboundHandler，也把处理操作交给下一个ChannelHandler，那么就会到 TailContext，在TailContext会提供一些默认处理
```
protected void onUnhandledInboundMessage(Object msg) {
    try {
        logger.debug(
              "Discarded inbound message {} that reached at the tail of the pipeline." + 
                "Please check your pipeline configuration.", msg);
    } finally {
        ReferenceCountUtil.release(msg);
    } 
}
```
比如channelRead中的onUnhandledInboundMessage方法，会把msg资源回收，防止内存泄露。  
强调一点的是，如果要执行整个链路，必须通过调用Channel方法触发，ChannelHandlerContext引用了 ChannelPipeline，所以也能间接操作channel的方法，但是会从当前ChannelHandlerContext绑定的 ChannelHandler作为起点开始，而不是ChannelHandlerContext的头和尾这个特性在不需要调用整个链路的情况下可以使用，可以增加一些效率。  
  
上述组件之间的关系  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Context.jpeg)  
  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/Context%E9%93%BE%E6%9D%A1.jpeg)  
  
>1. 每个Channel会绑定一个ChannelPipeline，ChannelPipeline中也会持有Channel的引用  
>2. ChannelPipeline持有ChannelHandlerContext链路，保留ChannelHandlerContext的头尾节点指针  
>3. 每个ChannelHandlerContext会对应一个ChannelHandler，也就相当于ChannelPipeline持有ChannelHandler链路  
>4. ChannelHandlerContext同时也会持有ChannelPipeline引用，也就相当于持有Channel引用  
>5. ChannelHandler链路会根据Handler的类型，分为InBound和OutBound两条链路  
  
### 源码分析之客户端启动  
  
#### 创建Channel  
在讲解 Netty 客户端程序时候我们提到指定 NioSocketChannel 用于创建客户端 NIO 套接字通道的实例，下面我们来看 NioSocketChannel 是如何创建一个 Java NIO 里面的 SocketChannel 的。首先我们来看 NioSocketChannel 的构造函数:
```
public NioSocketChannel() { 
    this(DEFAULT_SELECTOR_PROVIDER);
}
```
其中 DEFAULT_SELECTOR_PROVIDER 定义如下:
```
private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();
```
然后继续看
```
//这里的 provider 为 DEFAULT_SELECTOR_PROVIDER
public NioSocketChannel(SelectorProvider provider) {
    this(newSocket(provider)); 
}
```
其中 newSocket 代码如下:
```
private static SocketChannel newSocket(SelectorProvider provider) {
    try {
        return provider.openSocketChannel();
    } catch (IOException e) {
        throw new ChannelException("Failed to open a socket.", e);
    }
}
```
所以 NioSocketChannel 内部是管理一个客户端的 SocketChannel 的，这个 SocketChannel 就是讲 Java NIO 时候的 SocketChannel，也就是创建 NioSocketChannel 实例对象时候相当于执行了 Java NIO 中:
```
SocketChannel socketChannel = SocketChannel.open();
```
另外在 NioSocketChannel 的父类 AbstractNioChannel 的构造函数里面默认会记录队 op_read 事件感兴趣，这个后面当链接完成后会使用到
```
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) {
      super(parent, ch, SelectionKey.OP_READ);
}
```
另外在 NioSocketChannel 的父类 AbstractNioChannel 的构造函数里面还设置了该套接字为非阻塞的
```
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent); 
    this.ch = ch; 
    this.readInterestOp = readInterestOp; 
    try {
        ch.configureBlocking(false); 
    } catch (IOException e) {
      ... 
    }
}
```
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%88%9B%E5%BB%BAChannel.jpeg)  
  
整个流程涉及到 NioServerSocketChannel 的父类们。类图如下  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/NioServerSocketChannel%E7%88%B6%E7%B1%BB%E4%BB%AC.jpeg)  
  
>1.NioServerSocketChannel  
2.AbstractNioMessageChannel  
3.AbstractNioChannel  
4.AbstractChannel  
  
#### 注册Channel到Selector
下面我们看 Netty 里面是哪里创建的 NioSocketChannel 实例，哪里注册到选择器的。  
下面我们看下 Bootstrap 的 connect 操作代码:  
```
public ChannelFuture connect(InetAddress inetHost, int inetPort) {
    return connect(new InetSocketAddress(inetHost, inetPort));
}
```
类似 Java NIO 传递了一个 InetSocketAddress 对象用来记录服务端 ip 和端口:
```
public ChannelFuture connect(SocketAddress remoteAddress) {
    ...
    return doResolveAndConnect(remoteAddress, config.localAddress());
}
```
下面我们看下 doResolveAndConnect 的代码:
```
private ChannelFuture doResolveAndConnect(final SocketAddress remoteAddress, final SocketAddress localAddress) {
    //(1)
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    if (regFuture.isDone()) {
        if (!regFuture.isSuccess()) {
            return regFuture;
        }
        //(2)
        return doResolveAndConnect0(channel, remoteAddress, localAddress, channel.newPromise());
    }
    ...
    } 
}
```         
首先我们来看代码(1)initAndRegister:
```
final ChannelFuture initAndRegister() { 
    Channel channel = null;
    try {
    //(1.1)
    channel = channelFactory.newChannel();
    //(1.2) init(channel);          
    } catch (Throwable t) {
        ...
    }
    //(1.3)
    ChannelFuture regFuture = config().group().register(channel);
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) { 
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    } 
}
```                               
其中(1.1)作用就是创建一个 NioSocketChannel 的实例，代码(1.2)是具体设置内部套接字的选项的。  
代码(1.3)则是具体注册客户端套接字到选择器的，其首先会调用 NioEventLoop 的 register 方法，最后调用 NioSocketChannelUnsafe 的 register 方法:
```
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    ...
    AbstractChannel.this.eventLoop = eventLoop;
    if (eventLoop.inEventLoop()) { 
        register0(promise);
    } else { 
        try {
            eventLoop.execute(new Runnable() {
                @Override
                public void run() { 
                    register0(promise);
                } 
            });
        } catch (Throwable t) { ‘
                ...
        } 
    }
}
``` 
其中 register0 内部调用 doRegister，其代码如下:
```
protected void doRegister() throws Exception {
    boolean selected = false; 
    for (;;) {
        try {   
            //注册客户端 socket 到当前 eventloop 的 selector 上
            selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this); 
            return;
        } catch (CancelledKeyException e) {
            ...
        }
    } 
}
```
#### Connect
到这里代码(1)initAndRegister 的流程讲解完毕了，下面我们来看代码(2)的
```
public final void connect(final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    ... 
    try {
        ...
        boolean wasActive = isActive();
        if (doConnect(remoteAddress, localAddress)) {
           fulfillConnectPromise(promise, wasActive);
       } else {
            ...
        }
    } catch (Throwable t) {
          ... 
    }
}
```
其中 doConnect 代码如下:
```
protected boolean doConnect(SocketAddress remoteAddress, SocketAddress localAddress) throws Exception {
    ...
    boolean success = false; 
    try {
        //2.1
        boolean connected = SocketUtils.connect(javaChannel(), remoteAddress); 
        //2.2
        if (!connected) {
            selectionKey().interestOps(SelectionKey.OP_ CONNECT);
        }
        success = true; 
        return connected;
     } finally {
          if (!success) {
              doClose(); 
          }
     } 
}
```
其中 2.1 具体调用客户端套接字的 connect 方法，等价 于 Java NIO 里面的。  
代码 2.2 由于 connect 方法是异步的，所以类似 JavaNIO 调用 connect 方法进行判断，如果当前没有完成链接则设置对 op_connect 感兴趣。  
最后一个点就是何处进行的从选择器获取就绪的事件的,具体是在该客户端套接关联的 NioEventLoop 里面的做的，每个 NioEventLoop 里面有一个线程用来循环从选择器里面获取就绪的事件，然后进行处理:
```
protected void run() { 
    for (;;) {
        try { 
            ...
            select(wakenUp.getAndSet(false)); 
            ...
            processSelectedKeys();
            ...
        } catch (Throwable t) {
            handleLoopException(t); 
        }
        ...
     } 
}
```
其中 select 代码如下:
```
private void select(boolean oldWakenUp) throws IOException {
    Selector selector = this.selector; 
    try {
        ...
        for (;;) {
            ...
            int selectedKeys = selector.select(timeoutMillis);
            selectCnt ++;
            ...
         }
    } catch (CancelledKeyException e) {
        ... 
    }
}
```
可知会从选择器选取就绪的事件，其中 processSelectedKeys 代码如下:
```
private void processSelectedKeys() { 
    ...
    processSelectedKeysPlain(selector.selectedK eys());
    ... 
}
```
可知会获取已经就绪的事件集合，然后交给 processSelectedKeysPlain 处理，后者循环调用 processSelectedKey 具体处理每个事件，代码如下:
```
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    ... 
    try {
        //(3)如果是 op_connect 事件
        int readyOps = k.readyOps(); 
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops); 
            //3.1 
            unsafe.finishConnect();
        } 
        //4
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            ch.unsafe().forceFlush(); }
        //5
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read(); 
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise()); }
    }
```                                   
代码(3)如果当前事件 key 为 op_connect 则去掉 op_connect，然后调用 NioSocketChannel 的 doFinishConnect:
```
protected void doFinishConnect() throws Exception {
    if (!javaChannel().finishConnect())
        throw new Error(); 
    }
}
```
可知是调用了客户端套接字的 finishConnect 方法，最后会 调用 NioSocketChannel 的 doBeginRead 方法设置对 op_read 事件感兴趣:
```
protected void doBeginRead() throws Exception {
    ...
    final int interestOps = selectionKey.interestOps();
    if ((interestOps & readInterestOp) == 0) {
        selectionKey.interestOps(interestOps | readInterestOp);
    } 
}
```
这里 interestOps 为 op_read,上面在讲解 NioSocketChannel 的构造函数时候提到过。
代码(5)如果当前是 op_accept 事件说明是服务器监听套接字获取到了一个链接套接字，如果是 op_read,则说明可 以读取客户端发来的数据了，如果是后者则会激活管线里面的所有 handler 的 channelRead 方法，这里会激活我们自定义的 NettyClientHandler 的 channelRead 读取客户端发来的数据，然后在向客户端写入数据。  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/connect%E6%B5%81%E7%A8%8B.jpeg)  
  
### 源码分析之服务端启动   
  
#### bind核⼼流程  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/bind%E6%B5%81%E7%A8%8B.jpeg)  
                              
 >doBind()   
 >>initAndRegister   
   
 >doBind0()   
 >initAndRegister()   
 >>init()   
   
 >beginRead()
   
#### 创建Channel对象  
newChannel 在客户端中已经讲过。  
  
#### 初始化Channel配置    
abstract void init(Channel channel) throws Exception;   
用来将option()方法设置的属性进行初始化。  
  
#### 注册Channel到EventLoopGroup  
  
EventLoopGroup#register(Channel channel) ⽅法，注册 Channel 到 EventLoopGroup 中。整体流程如下:  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E6%B3%A8%E5%86%8C%E6%B5%81%E7%A8%8B.jpeg)  
  
register:  
AbstractUnsafe#register(EventLoop eventLoop, final ChannelPromise promise)方法开始校验传⼊的 eventLoop 参数⾮空。  
调⽤ #isRegistered() 方法。  
  
register0  
#register0(ChannelPromise promise) 方法中调用doregister()方法。doregister()方法中包含了register(Selector sel, int ops, Object att)方法。   
SelectableChannel#register(Selector sel, int ops, Object att) ⽅法，注册 Java 原生 NIO 的 Channel 对象到 Selector 对象上。但是为什么感兴趣的事件是为 0 呢？正常情况下，对于服务端来说，需要注册 SelectionKey.OP_ACCEPT 事件。  
>1.注册方式是多态的，它既可以被 NIOServerSocketChannel 用来监听客户端的连接入，也可以注册 SocketChannel ⽤来监听⽹络读或者写操作。  
2.通过 SelectionKey#interestOps(int ops) ⽅法可以⽅便地修改监听操作位。所以此处注册需要获取 SelectionKey 并给 AbstractNIOChannel 的成员变量量 selectionKey 赋值。    
  
execute  
#execute(Runnable task) ⽅法，执行⼀个任务。  
```
 1: @Override
 2: public void execute(Runnable task) {
 3:     if (task == null) {
 4:         throw new NullPointerException("task");
 5:     }
 6:
 7:     // 获得当前是否在 EventLoop 的线程中
 8:     boolean inEventLoop = inEventLoop(); 
 9:     // 添加到任务队列列
10:     addTask(task);
11:     if (!inEventLoop) {
12:         // 创建线程 
13:         startThread();
14:         // 若已经关闭，移除任务，并进⾏行行拒绝
15:         if (isShutdown() && removeTask(task)) {
16:             reject(); 
17:         }
18:     }
19:
20:     // 唤醒线程
21:     if (!addTaskWakesUp && wakesUpForTask(task)) {
22:         wakeup(inEventLoop);
23:     }
24: }
```    

### 源码分析之EventLoop  
  
NioEventLoopGroup  
一个 Netty 程序启动时，至少要指定一个 EventLoopGroup(如果使用到的是 NIO, 那么通常是 NioEventLoopGroup), 那么这个 NioEventLoopGroup 在 Netty 中到底扮演着什么角色呢? 我们知道, Netty 是 Reactor 模型的一个实现, 那么首先从 Reactor 的线程模型开始吧。  

#### 关于Reactor的线程模型  
首先我们再次来看一下 Reactor 的线程模型。  
Reactor 的线程模型有三种:  
>单线程模型  
多线程模型  
主从多线程模型  
  
单线程模型  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpeg)  
  
所谓单线程,即 acceptor 处理和 handler 处理都在一个线程中处理. 这个模型的坏处显而易见: 当其中某个 handler 阻塞时, 会导致其他所有的 client 的 handler 都得不到执行, 并且更严重的是, handler 的阻塞也会导致整个服务不能接收新的 client 请求(因为 acceptor 也被阻塞了). 因为有这么多的缺陷, 因此单线程Reactor 模型用的比较少.  
  
那么什么是多线程模型呢? Reactor 的多线程模型与单线程模型的区别就是 acceptor 是一个单独的线程处理, 并且有一组特定的 NIO 线程来负责各个客户端连接的 IO 操作.Reactor 多线程模型如下:  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpeg)  
  
Reactor 多线程模型有如下特点:
>有专门一个线程, 即 Acceptor 线程用于监听客户端的TCP连接请求.  
客户端连接的 IO 操作都是由一个特定的 NIO 线程池负责. 每个客户端连接都与一个特定的 NIO 线程绑定, 因此在这个客户端连接中的所有 IO 操作都是在同一个线程中完成的.  
客户端连接有很多, 但是 NIO 线程数是比较少的, 因此一个 NIO 线程可以同时绑定到多个客户端连接中.  
  
接下来我们再来看一下 Reactor 的主从多线程模型. 一般情况下, Reactor 的多线程模式已经可以很好的工作了, 但是我们考虑一下如下情况: 如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时, 进行一些权限的检查, 那么单线程的 Acceptor 很有可能就处理不过来, 造成了大量的客户端不能连接到服务器. Reactor 的主从多线程模型就是在这样的情况下提出来的, 它的特点是: 服务器端接收客户端的连接请求不再是一个线程, 而是由一个独立的线程池组成. 它的线程模型如下  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E4%B8%BB%E4%BB%8E%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpeg)  
  
可以看到, Reactor 的主从多线程模型和 Reactor 多线程模型很类似, 只不过 Reactor 的主从多线程模型的 acceptor 使用了线程池来处理大量的客户端请求.  

#### NioEventLoopGroup与Reactor线程模型的对应  
我们介绍了三种 Reactor 的线程模型, 那么它们和 NioEventLoopGroup 又有什么关系呢? 其实, 不同的设置 NioEventLoopGroup 的方式就对应了不同的 Reactor 的线程模型.   
  
单线程模型  
来看一下下面的例子:  
```
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup)
 .channel(NioServerSocketChannel.class)
 ...
 ```
注意, 我们实例化了一个 NioEventLoopGroup, 构造器参数是1, 表示 NioEventLoopGroup 的线程池大小是1. 然后接着我们调用 b.group(bossGroup) 设置了服务器端的 EventLoopGroup. 有些朋友可能会有疑惑: 我记得在启动服务器端的 Netty 程序时, 是需要设置 bossGroup 和 workerGroup 的, 为什么这里就只有一个 bossGroup? 其实 很简单, ServerBootstrap 重写了 group 方法:  
```
@Override
public ServerBootstrap group(EventLoopGroup group) {
    return group(group, group);
}
```
因此当传入一个 group 时, 那么 bossGroup 和 workerGroup 就是同一个 NioEventLoopGroup 了. 这时候呢, 因为 bossGroup 和 workerGroup 就是同一个 NioEventLoopGroup, 并且这个 NioEventLoopGroup 只有一个线程, 这 样就会导致 Netty 中的 acceptor 和后续的所有客户端连接的 IO 操作都是在一个线程中处理的. 那么对应到 Reactor 的线程模型中, 我们这样设置 NioEventLoopGroup 时, 就相当于 Reactor 单线程模型.  
  
多线程模型   
同理, 再来看一下下面的例子: 
```
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 ...
```
bossGroup 中只有一个线程, 而 workerGroup 中的线程是 CPU 核心数乘以2, 因此对应的到 Reactor 线程模型中, 我们知道, 这样设置的 NioEventLoopGroup 其实就是 Reactor 多线程模型.  
  
主从多线程模型
```  
EventLoopGroup bossGroup = new NioEventLoopGroup(4);
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 ...
```
bossGroup 线程池中的线程数我们设置为4, 而 workerGroup 中的线程是 CPU 核心数乘以2, 因此对应的到 Reactor 线程模型中, 我们知道, 这样设置的 NioEventLoopGroup 其实就是 Reactor 主从多线程模型.  
  
Netty 的服务器端的 acceptor 阶段, 没有使用到多线程, 因此上面的 主从多线程模型 在 Netty 的服务器端是不存在的.  
  
服务器端的 ServerSocketChannel 只绑定到了 bossGroup 中的一个线程, 因此在调用 Java NIO 的 Selector.select 处理客户端的连接请求时, 实际上是在一个线程中的, 所以对只有一个服务的应用来说, bossGroup 设置多个线程是 没有什么作用的, 反而还会造成资源浪费.  
  
#### NioEventLoopGroup  
  
#### NioEventLoopGroup类层次结构  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/NioEventLoopGroup%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpeg)  
  
#### NioEventLoopGroup实例化过程
>EventLoopGroup(其实是MultithreadEventExecutorGroup) 内部维护一个类型为 EventExecutor children 数组, 其大小是 nThreads, 这样就构成了一个线程池  
如果我们在实例化 NioEventLoopGroup 时, 如果指定线程池大小, 则 nThreads 就是指定的值, 反之是处理器 核心数 * 2  
MultithreadEventExecutorGroup 中会调用 newChild 抽象方法来初始化 children 数组  
抽象方法 newChild 是在 NioEventLoopGroup 中实现的, 它返回一个 NioEventLoop 实例.  
NioEventLoop 属性:  
SelectorProvider provider 属性: NioEventLoopGroup 构造器中通过 SelectorProvider.provider() 获取一个 SelectorProvider  
Selector selector 属性: NioEventLoop 构造器中通过调用通过 selector = provider.openSelector() 获取一个 selector 对象.  
  
#### NioEventLoop  
NioEventLoop 继承于 SingleThreadEventLoop, 而 SingleThreadEventLoop 又继承于 SingleThreadEventExecutor. SingleThreadEventExecutor 是 Netty 中对本地线程的抽象, 它内部有一个 Thread thread 属性, 存储了一个本地 Java 线程. 因此我们可以认为, 一个 NioEventLoop 其实和一个特定的线程绑定, 并且 在其生命周期内, 绑定的线程都不会再改变.  
  
#### NioEventLoop类层次结构  
![](https://github.com/YufeizhangRay/image/blob/master/Netty/NioEventLoop%20%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpeg)  
        
 NioEventLoop 的类层次结构图还是比较复杂的, 不过我们只需要关注几个重要的点即可. 首先 NioEventLoop 的继
链如承链如下:
```
NioEventLoop -> SingleThreadEventLoop -> SingleThreadEventExecutor -> AbstractScheduledEventExecutor
```
在 AbstractScheduledEventExecutor 中, Netty 实现了 NioEventLoop 的 schedule 功能, 即我们可以通过调用一个 NioEventLoop 实例的 schedule 方法来运行一些定时任务. 而在 SingleThreadEventLoop 中, 又实现了任务队列的功能, 通过它, 我们可以调用一个 NioEventLoop 实例的 execute 方法来向任务队列中添加一个 task, 并由 NioEventLoop 进行调度执行.  
  
通常来说, NioEventLoop 肩负着两种任务, 第一个是作为 IO 线程, 执行与 Channel 相关的 IO 操作, 包括调用 select 等待就绪的 IO 事件、读写数据与数据的处理等; 而第二个任务是作为任务队列, 执行 taskQueue 中的任务, 例如用户调用 eventLoop.schedule 提交的定时任务也是这个线程执行的.  
  
#### NioEventLoop的实例化过程  
SingleThreadEventExecutor 有一个名为 thread 的 Thread 类型字段, 这个字段就代表了与SingleThreadEventExecutor 关联的本地线程. 下面是这个构造器的代码:  
```
protected SingleThreadEventExecutor(EventExecutorGroup parent, ThreadFactory threadFactory, boolean addTaskWakesUp){
    this.parent = parent;
    this.addTaskWakesUp = addTaskWakesUp;
    thread = threadFactory.newThread(new Runnable() {
        @Override
        public void run() {
            boolean success = false;
            updateLastExecutionTime();
            try {
                SingleThreadEventExecutor.this.run();
                success = true;
            } catch (Throwable t) {
                logger.warn("Unexpected exception from an event executor: ", t);
            } finally {
                // 省略清理代码
                ...  
            }
            threadProperties = new DefaultThreadProperties(thread);
            taskQueue = newTaskQueue();
        }
    });
} 
```
在 SingleThreadEventExecutor 构造器中, 通过 threadFactory.newThread 创建了一个新的 Java 线程. 在这个线程中所做的事情主要就是调用 SingleThreadEventExecutor.this.run() 方法, 而因为 NioEventLoop 实现了这个方法, 因此根据多态性, 其实调用的是 NioEventLoop.run() 方法.  

#### EventLoop与Channel的关联  
从之前的时序图中我们可以看到, 当调用了 AbstractChannel#AbstractUnsafe.register 后, 就完成了 Channel 和 EventLoop 的关联. register 实现如下:
```
@Override
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    // 删除条件检查.
    ...
    AbstractChannel.this.eventLoop = eventLoop;
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else { 
        try {
            eventLoop.execute(new OneTimeTask() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            ...
        } 
    }
}
```
在 AbstractChannel#AbstractUnsafe.register 中, 会将一个 EventLoop 赋值给 AbstractChannel 内部的 eventLoop 字段, 到这里就完成了 EventLoop 与 Channel 的关联过程.  
  
#### EventLoop的启动  
在前面我们已经知道了, NioEventLoop 本身就是一个 SingleThreadEventExecutor, 因此 NioEventLoop 的启动, 其实就是 NioEventLoop 所绑定的本地 Java 线程的启动. 依照这个思想, 我们只要找到在哪里调用了 SingleThreadEventExecutor 的 thread 字段的 start() 方法就可以知道是在哪里启动的这个线程了. 从代码中搜索, thread.start() 被封装到 SingleThreadEventExecutor.startThread() 方法中了:  
```
private void startThread() {
    if (STATE_UPDATER.get(this) == ST_NOT_STARTED) {
        if (STATE_UPDATER.compareAndSet(this, ST_NOT_STARTED, ST_STARTED)) {
            thread.start();
        } 
    }
}
```
STATE_UPDATER 是 SingleThreadEventExecutor 内部维护的一个属性, 它的作用是标识当前的 thread 的状态. 在初始的时候, STATE_UPDATER == ST_NOT_STARTED , 因此第一次调用 startThread() 方法时, 就会进入到 if 语句内, 进而调用到 thread.start(). 而这个关键的 startThread() 方法又是在哪里调用的呢? 经过方法调用关系搜索, 我们发现, startThread 是在 SingleThreadEventExecutor.execute 方法中调用的:  
```
@Override
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }
    boolean inEventLoop = inEventLoop();
    if (inEventLoop) {
        addTask(task);
    } else {
        startThread(); // 调用 startThread 方法, 启动EventLoop 线程. addTask(task);
        if (isShutdown() && removeTask(task)) {
            reject(); 
        }
    }
    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    }
}
```  
既然如此,那现在我们的工作就变为了寻找在哪里第一次调用了 SingleThreadEventExecutor.execute() 方法. 如果留心的读者可能已经注意到了, 我们在 EventLoop 与 Channel 的关联这一小节时, 有提到到在注册 channel 的过程中, 会在 AbstractChannel#AbstractUnsafe.register 中调用 eventLoop.execute 方法, 在 EventLoop 中 进行 Channel 注册代码的执行, AbstractChannel#AbstractUnsafe.register 部分代码如下:
```
if (eventLoop.inEventLoop()) {
    register0(promise);
} else { 
    try {
        eventLoop.execute(new OneTimeTask() {
            @Override
            public void run() {
                register0(promise);
            } 
        });
    } catch (Throwable t) {
        ...
    } 
}
```
很显然, 一路从 Bootstrap.bind 方法跟踪到 AbstractChannel#AbstractUnsafe.register 方法, 整个代码都是在主线程中运行的, 因此上面的 eventLoop.inEventLoop() 就为 false, 于是进入到 else 分支, 在这个分支中调用了 eventLoop.execute. eventLoop 是一个 NioEventLoop 的实例, 而 NioEventLoop 没有实现 execute 方法, 因此调用的是 SingleThreadEventExecutor.execute:
```
@Override
public void execute(Runnable task) {
    ...
    boolean inEventLoop = inEventLoop();
    if (inEventLoop) {
        addTask(task);
    } else {
        startThread();
        addTask(task);
        if (isShutdown() && removeTask(task)) {
            reject(); 
        }
    }
    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    } 
}
```
我们已经分析过了, inEventLoop == false, 因此执行到 else 分支, 在这里就调用了 startThread() 方法来启动 SingleThreadEventExecutor 内部关联的 Java 本地线程了. 总结一句话, 当 EventLoop.execute 第一次被调用时, 就会触发 startThread() 的调用, 进而导致了 EventLoop 所对应的 Java 线程的启动.   
  
### 源码分析之IO处理循环  
接下来我们先从 IO 操纵方面入手, 看一下 TCP 数据是如何从 Java NIO Socket 传递到我们的 handler 中的.  
Netty 是 Reactor 模型的一个实现, 并且是基于 Java NIO 的, 那么从 Java NIO 的前生今世 之四 NIO Selector 详解 中我们知道, Netty 中必然有一个 Selector 线程, 用于不断调用 Java NIO 的 Selector.select 方法, 查询当前是否有就绪的 IO 事件. 回顾一下在 Java NIO 中所讲述的 Selector 的使用流程:  
>1. 通过 Selector.open() 打开一个 Selector.  
>2. 将 Channel 注册到 Selector 中, 并设置需要监听的事件(interest set)  
>3. 不断重复:  
>>调用 select() 方法  
调用 selector.selectedKeys() 获取 selected keys   
迭代每个 selected key:  
>>>从 selected key 中获取 对应的 Channel 和附加信息(如果有的话)  
判断是哪些 IO 事件已经就绪了, 然后处理它们. 如果是 OP_ACCEPT 事件, 则调用 "SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept()" 获取 SocketChannel, 并将它 设置为 非阻塞的, 然后将这个 Channel 注册到 Selector 中.   
根据需要更改 selected key 的监听事件.  
将已经处理过的 key 从 selected keys 集合中删除.  
上面的使用流程用代码来体现就是:  
  
```
public class NioEchoServer {
    private static final int BUF_SIZE = 256;
    private static final int TIMEOUT = 3000;
    public static void main(String args[]) throws Exception {
    // 打开服务端 Socket
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    
    // 打开 Selector
    Selector selector = Selector.open();
    
    // 服务端 Socket 监听8080端口, 并配置为非阻塞模式 
    serverSocketChannel.socket().bind(new InetSocketAddress(8080)); 
    serverSocketChannel.configureBlocking(false);
    
    // 将 channel 注册到 selector 中.
    // 通常我们都是先注册一个 OP_ACCEPT 事件, 然后在 OP_ACCEPT 到来时, 再将这个 Channel 的 OP_READ 注册到 Selector 中.
    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
    
    while (true) {
        // 通过调用 select 方法, 阻塞地等待 channel I/O 可操作 
        if (selector.select(TIMEOUT) == 0) {
            System.out.print(".");
            continue; 
        }
        
        // 获取 I/O 操作就绪的 SelectionKey, 通过 SelectionKey 可以知道哪些 Channel 的哪 类 I/O 操作已经就绪.
        Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
        
        while (keyIterator.hasNext()) {
            // 当获取一个 SelectionKey 后, 就要将它删除, 表示我们已经对这个 IO 事件进行了处 
            keyIterator.remove();
            
            SelectionKey key = keyIterator.next();
            
            if (key.isAcceptable()) {
                // 当 OP_ACCEPT 事件到来时, 我们就有从 ServerSocketChannel 中获取一个代表客户端的连接
                // 注意, 在 OP_ACCEPT 事件中, 从 key.channel() 返回的 Channel 是 ServerSocketChannel.
                // 而在 OP_WRITE 和 OP_READ 中, 从 key.channel() 返回的是 SocketChannel.
                SocketChannel clientChannel = ((ServerSocketChannel)key.channel()).accept();
                clientChannel.configureBlocking(false);
                //在 OP_ACCEPT 到来时, 再将这个 Channel 的 OP_READ 注册到 Selector 中.
                // 注意, 这里我们如果没有设置 OP_READ 的话, 即 interest set 仍然是 OP_CONNECT 的话, 那么 select 方法会一直直接返回.
                clientChannel.register(key.selector(), OP_READ, ByteBuffer.allocate(BUF_SIZE));
             }
             
             if (key.isReadable()) {
                 SocketChannel clientChannel = (SocketChannel) key.channel();
                 ByteBuffer buf = (ByteBuffer) key.attachment();
                 long bytesRead = clientChannel.read(buf);
                 if (bytesRead == -1) {
                     clientChannel.close();
                 } else if (bytesRead > 0) {
                    key.interestOps(OP_READ | SelectionKey.OP_WRITE);
                    System.out.println("Get data length: " + bytesRead);
                 }
              }
              if (key.isValid() && key.isWritable()) {
                  ByteBuffer buf = (ByteBuffer) key.attachment();
                  buf.flip();
                  SocketChannel clientChannel = (SocketChannel) key.channel();
                  clientChannel.write(buf);
                  if (!buf.hasRemaining()) {
                      key.interestOps(OP_READ);
                  }
                  buf.compact();
              }
           } 
        }
    } 
}
 
```
 
还记得不, 上面操作的第一步 通过 Selector.open() 打开一个 Selector 我们已经在第一章的 Channel 实例化这一小节中已经提到了, Netty 中是通过调用 SelectorProvider.openSocketChannel() 来打开一个新的 Java NIO SocketChannel:
```
private static SocketChannel newSocket(SelectorProvider provider) {
    ...
    return provider.openSocketChannel();
}
```
第二步 将 Channel 注册到 Selector 中, 并设置需要监听的事件(interest set) 的操作我们在第一章 channel 的注 册过程 中也分析过了, 我们在来回顾一下, 在客户端的 Channel 注册过程中, 会有如下调用链:
```
Bootstrap.initAndRegister ->
    AbstractBootstrap.initAndRegister ->
        MultithreadEventLoopGroup.register ->
            SingleThreadEventLoop.register ->
                AbstractUnsafe.register ->
                    AbstractUnsafe.register0 ->
                        AbstractNioChannel.doRegister
```
在 AbstractUnsafe.register 方法中调用了 register0 方法:
```
@Override
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    // 省略条件判断和错误处理 
    AbstractChannel.this.eventLoop = eventLoop; register0(promise);
}
```
register0 方法代码如下:
```
private void register0(ChannelPromise promise) {
    boolean firstRegistration = neverRegistered;
    doRegister();
    neverRegistered = false;
    registered = true;
    safeSetSuccess(promise);
    pipeline.fireChannelRegistered();
    // Only fire a channelActive if the channel has never been registered. This prevents firing
    // multiple channel actives if the channel is deregistered and re-registered.
    if (firstRegistration && isActive()) {
        pipeline.fireChannelActive();
    }
}
```
register0 又调用了 AbstractNioChannel.doRegister:
```
@Override
protected void doRegister() throws Exception {
    // 省略错误处理
    selectionKey = javaChannel().register(eventLoop().selector, 0, this);
}
```
在这里 javaChannel() 返回的是一个 Java NIO SocketChannel 对象, 我们将此 SocketChannel 注册到前面第一步获取的 Selector 中.
  
#### thread的run循环  
在 EventLoop 的启动 一小节中, 我们已经了解到了, 当 EventLoop.execute 第一次被调用时, 就会触发 startThread() 的调用, 进而导致了 EventLoop 所对应的 Java 线程的启动. 接着我们来更深入一些, 来看一下此线程启动后都会做什么东东吧. 下面是此线程的 run() 方法, 我已经把一些异常处理和收尾工作的代码都去掉了. 这个 run 方法可以说是十分简单, 主要就是调用了 SingleThreadEventExecutor.this.run() 方法. 而 SingleThreadEventExecutor.run() 是一个抽象方法, 它的实现在 NioEventLoop 中.  
```
thread = threadFactory.newThread(new Runnable() {
    @Override
    public void run() {
        boolean success = false;
        updateLastExecutionTime();
        try {
            SingleThreadEventExecutor.this.run();
            success = true;
        } catch (Throwable t) {
            logger.warn("Unexpected exception from an event executor: ", t);
        } finally {
            ...
        }
    }
});
```
继续跟踪到 NioEventLoop.run() 方法, 其源码如下:
```
@Override
protected void run() {
    for (;;) {
        boolean oldWakenUp = wakenUp.getAndSet(false);
        try {
            if (hasTasks()) {
                selectNow();
            } else {
                select(oldWakenUp);
                if (wakenUp.get()) {
                    selector.wakeup();
                }
            }
            cancelledKeys = 0;
            needsToSelectAgain = false;
            final int ioRatio = this.ioRatio;
            if (ioRatio == 100) {
                processSelectedKeys();
                runAllTasks();
            } else {
                final long ioStartTime = System.nanoTime();
                processSelectedKeys();
                final long ioTime = System.nanoTime() - ioStartTime;
                runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
            }
            if (isShuttingDown()) {
                closeAll();
                if (confirmShutdown()) {
                    break;
                } 
            }
        } catch (Throwable t) {
            ...
        } 
    }
}
```  
看到了上面代码的 for(;;) 所构成的死循环了没? 原来 NioEventLoop 事件循环的核心就是这里! 现在我们把上 面所提到的 Selector 使用步骤的第三步的部分也找到了. 这个 run 方法可以说是 Netty NIO 的核心, 属于重中之重, 把它分析明白了, 那么对 Netty 的事件循环机制也就了解了大部分了.    
  
#### IO事件的轮询  
首先, 在 run 方法中, 第一步是调用 hasTasks() 方法来判断当前任务队列中是否有任务:  
```
protected boolean hasTasks() {
    assert inEventLoop();
    return !taskQueue.isEmpty();
}
```
这个方法很简单, 仅仅是检查了一下 taskQueue 是否为空. 至于 taskQueue 是什么呢, 其实它就是存放一系列的需 要由此 EventLoop 所执行的任务列表. 关于 taskQueue, 我们这里暂时不表, 等到后面再来详细分析它. 当 taskQueue 不为空时, 就执行到了 if 分支中的 selectNow() 方法. 然而当 taskQueue 为空时, 执行的是 select(oldWakenUp) 方法. 那么 selectNow() 和 select(oldWakenUp) 之间有什么区别呢? 来看一下, selectNow() 的源码如下:
``` 
void selectNow() throws IOException {
    try {
        selector.selectNow();
    } finally {
        // restore wakup state if needed
        if (wakenUp.get()) {
        selector.wakeup();
        }
    } 
}

```
首先调用了 selector.selectNow() 方法, 这个 selector 字段正是 Java NIO 中的多路复用器 Selector. 那么这里 selector.selectNow() 就很好理解了, selectNow() 方法会检查当前是否有就绪的 IO 事件, 如果有, 则返回就绪 IO 事件的个数; 如果没有, 则返回0. 注意, selectNow() 是立即返回的, 不会阻塞当前线程. 当 selectNow() 调用后, finally 语句块中会检查 wakenUp 变量是否为 true, 当为 true 时, 调用 selector.wakeup() 唤醒 select() 的阻塞调用.  
  
看了 if 分支的 selectNow 方法后, 我们再来看一下 else 分支的 select(oldWakenUp) 方法. 其实 else 分支的 select(oldWakenUp) 方法的处理逻辑比较复杂, 而我们这里的目的暂时不是分析这个方法调用的具体工作, 因此这里长话短说, 只列出关注的内如:
```
private void select(boolean oldWakenUp) throws IOException {
    Selector selector = this.selector;
    try {
        ...
        int selectedKeys = selector.select(timeoutMillis);
        ...
    } catch (CancelledKeyException e) {
        ...
    } 
}
```
在这个 select 方法中, 调用了 selector.select(timeoutMillis), 而这个调用是会阻塞住当前线程的, timeoutMillis 是阻塞的超时时间. 到来这里, 我们可以看到, 当 hasTasks() 为真时, 调用的的 selectNow() 方法是不会阻塞当前线程的, 而当 hasTasks() 为假时, 调用的 select(oldWakenUp) 是会阻塞当前线程的. 这其实也很好理解: 当 taskQueue 中没有任务时, 那么 Netty 可以阻塞地等待 IO 就绪事件; 而当 taskQueue 中有任务时, 我们自然地希望所提交的任务可以尽快地执行, 因此 Netty 会调用非阻塞的 selectNow() 方法, 以保证 taskQueue 中的任务尽快可以执行.  
  
#### IO事件的处理  
在 NioEventLoop.run() 方法中, 第一步是通过 select/selectNow 调用查询当前是否有就绪的 IO 事件. 那么当有 IO 事件就绪时, 第二步自然就是处理这些 IO 事件啦. 首先让我们来看一下 NioEventLoop.run 中循环的剩余部分:  
```
final int ioRatio = this.ioRatio;
if (ioRatio == 100) {
    processSelectedKeys();
    runAllTasks();
} else {
    final long ioStartTime = System.nanoTime();
    processSelectedKeys();
    final long ioTime = System.nanoTime() - ioStartTime;
    runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
}
```
上面列出的代码中, 有两个关键的调用, 第一个是 processSelectedKeys() 调用, 根据字面意思, 我们可以猜出这个方法肯定是查询就绪的 IO 事件, 然后处理它; 第二个调用是 runAllTasks(), 这个方法我们也可以一眼就看出来它的功能就是运行 taskQueue 中的任务. 这里的代码还有一个十分有意思的地方, 即 ioRatio. 那什么是 ioRatio呢? 它 表示的是此线程分配给 IO 操作所占的时间比(即运行 processSelectedKeys 耗时在整个循环中所占用的时间). 例如 ioRatio 默认是 50, 则表示 IO 操作和执行 task 的所占用的线程执行时间比是 1 : 1. 当知道了 IO 操作耗时和它所占用的时间比, 那么执行 task 的时间就可以很方便的计算出来了:
```
设 IO 操作耗时为 ioTime, ioTime 占的时间比例为 ioRatio, 则: ioTime / ioRatio = taskTime / taskRatio
taskRatio = 100 - ioRatio
=> taskTime = ioTime * (100 - ioRatio) / ioRatio
```
根据上面的公式, 当我们设置 ioRate = 70 时, 则表示 IO 运行耗时占比为70%, 即假设某次循环一共耗时为 100ms, 那么根据公式, 我们知道 processSelectedKeys() 方法调用所耗时大概为70ms(即 IO 耗时), 而 runAllTasks() 耗时大概为 30ms(即执行 task 耗时). 当 ioRatio 为 100 时, Netty 就不考虑 IO 耗时的占比, 而是分别调用 processSelectedKeys()、runAllTasks(); 而当 ioRatio 不为 100时, 则执行到 else 分支, 在这个分支中, 首先记录下 processSelectedKeys() 所执行的时间(即 IO 操作的耗时), 然后根据公式, 计算出执行 task 所占用的时间, 然后以此为参数, 调用 runAllTasks().  
我们这里先分析一下 processSelectedKeys() 方法调用, runAllTasks() 我们留到下一节再分析. processSelectedKeys() 方法的源码如下:
```
private void processSelectedKeys() {
    if (selectedKeys != null) {
        processSelectedKeysOptimized(selectedKeys.flip());
    } else {
        processSelectedKeysPlain(selector.selectedKeys());
    }
}
```
这个方法中, 会根据 selectedKeys 字段是否为空, 而分别调用 processSelectedKeysOptimized 或 processSelectedKeysPlain. selectedKeys 字段是在调用 openSelector() 方法时, 根据 JVM 平台的不同, 而有设置不同的值, 在我所调试这个值是不为 null 的. 其实 processSelectedKeysOptimized 方法 processSelectedKeysPlain 没有太大的区别, 为了简单起见, 我们以 processSelectedKeysOptimized 为例分析一下源码的工作流程吧.
```
private void processSelectedKeysOptimized(SelectionKey[] selectedKeys) {
    for (int i = 0;; i ++) {
        final SelectionKey k = selectedKeys[i];
        if (k == null) {
            break;
        }
        selectedKeys[i] = null;
        final Object a = k.attachment();
        if (a instanceof AbstractNioChannel) {
            processSelectedKey(k, (AbstractNioChannel) a);
        } else {
            @SuppressWarnings("unchecked")
            NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
            processSelectedKey(k, task);
        }
        ...
    }
}
```
其实你别看它代码挺多的, 但是关键的点就两个: 迭代 selectedKeys 获取就绪的 IO 事件, 然后为每个事件都调用 processSelectedKey 来处理它. 这里正好完美对应上了我们提到的 Selector 的使用流程中的第三步里操作. 还有一点需要注意的是, 我们可以调用 selectionKey.attach(object) 给一个 selectionKey 设置一个附加的字段, 然后可以通过 Object attachedObj = selectionKey.attachment() 获取它. 上面代代码正是通过了 k.attachment() 来获取一个附加在 selectionKey 中的对象, 那么这个对象是什么呢? 它又是在哪里设置的呢? 我们再来回忆一下 SocketChannel 是如何注册到 Selector 中的: 在客户端的 Channel 注册过程中, 会有如下调用链:
```
  Bootstrap.initAndRegister ->
    AbstractBootstrap.initAndRegister ->
        MultithreadEventLoopGroup.register ->
            SingleThreadEventLoop.register ->
                AbstractUnsafe.register ->
                    AbstractUnsafe.register0 ->
                        AbstractNioChannel.doRegister
 ```
最后的 AbstractNioChannel.doRegister 方法会调用 SocketChannel.register 方法注册一个 SocketChannel 到指定的 Selector:
```
@Override
protected void doRegister() throws Exception {
    // 省略错误处理
    selectionKey = javaChannel().register(eventLoop().selector, 0, this);
}
```
特别注意一下 register 的第三个参数, 这个参数是设置 selectionKey 的附加对象的, 和调用 selectionKey.attach(object) 的效果一样. 而调用 register 所传递的第三个参数是 this, 它其实就是一个 NioSocketChannel 的实例. 那么这里就很清楚了, 我们在将 SocketChannel 注册到 Selector 中时, 将 SocketChannel 所对应的 NioSocketChannel 以附加字段的方式添加到了selectionKey 中. 再回到 processSelectedKeysOptimized 方法中, 当我们获取到附加的对象后, 我们就调用 processSelectedKey 来处理 这个 IO 事件:  
```
final Object a = k.attachment();
if (a instanceof AbstractNioChannel) {
    processSelectedKey(k, (AbstractNioChannel) a);
} else {
    @SuppressWarnings("unchecked")
    NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
    processSelectedKey(k, task);
}
```
processSelectedKey 方法源码如下:
```
private static void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final NioUnsafe unsafe = ch.unsafe();
    ...
    try {
        int readyOps = k.readyOps(); 
        // 可读事件
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
            if (!ch.isOpen()) {
                // Connection already closed - no need to handle write.
                return;
            }
        }
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
            if (!ch.isOpen()) {
                // Connection already closed - no need to handle write.
                return; 
            }
        }
        // 可写事件
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
            ch.unsafe().forceFlush();
        }
        // 连接建立事件
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);
            unsafe.finishConnect();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    } 
}
```
这个代码是 Java NIO 的 Selector 的那一套处理流程. rocessSelectedKey 中处理了三个 事件, 分别是:  
>OP_READ, 可读事件, 即 Channel 中收到了新数据可供上层读取.  
OP_WRITE, 可写事件, 即上层可以向 Channel 写入数据.  
OP_CONNECT, 连接建立事件, 即 TCP 连接已经建立, Channel 处于 active 状态.  
  
下面我们分别根据这三个事件来看一下 Netty 是怎么处理的吧.   
  
OP_READ 处理  
当就绪的 IO 事件是 OP_READ, 代码会调用 unsafe.read() 方法, 即:  
```
// 可读事件
if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
    unsafe.read();
    if (!ch.isOpen()) {
        // Connection already closed - no need to handle write.
        return; 
    }
}
```  
unsafe 是一个 NioSocketChannelUnsafe 实例, 负责的是 Channel 的底层 IO 操作. 我们可以利用 Intellij IDEA 提供的 Go To Implementations 功能, 寻找到这个方法的实现. 最后我们发现这个方法没有在 NioSocketChannelUnsafe 中实现, 而是在它的父类 AbstractNioByteChannel 实现的, 它的实现源码如下:
 ```
@Override
public final void read() {
    ...
    ByteBuf byteBuf = null;
    int messages = 0;
    boolean close = false;
    try {
        int totalReadAmount = 0;
        boolean readPendingReset = false;
        do {
            byteBuf = allocHandle.allocate(allocator);
            int writable = byteBuf.writableBytes();
            int localReadAmount = doReadBytes(byteBuf);
            // 检查读取结果. ...
            pipeline.fireChannelRead(byteBuf);
            byteBuf = null;
            ...
            totalReadAmount += localReadAmount;
            // 检查是否是配置了自动读取, 如果不是, 则立即退出循环.
            ...
        } while (++ messages < maxMessagesPerRead);
        pipeline.fireChannelReadComplete();
        allocHandle.record(totalReadAmount);
        if (close) {
            closeOnRead(pipeline);
            close = false;
        }
    } catch (Throwable t) {
        handleReadException(pipeline, byteBuf, t, close);
    } finally {
    } 
}
```
read() 源码比较长, 我为了篇幅起见, 删除了部分代码, 只留下了主干. 不过我建议读者朋友们自己一定要看一下 read() 源码, 这对理解 Netty 的 EventLoop 十分有帮助. 上面 read 方法其实归纳起来, 可以认为做了如下工作:
>1. 分配 ByteBuf  
>2. 从 SocketChannel 中读取数据  
>3. 调用 pipeline.fireChannelRead 发送一个 inbound 事件.  
  
前面两点没什么好说的, 第三点 pipeline.fireChannelRead 读者朋友们看到了有没有会心一笑地感觉呢? 反正我看到这里时是有的.   pipeline.fireChannelRead 是 inbound 事件起点. 当调用了 pipeline.fireIN_EVT() 后, 那么就产生了一 个 inbound 事件, 此事件会以 head -> customContext -> tail 的方向依次流经 ChannelPipeline 中的各个 handler. 调用了 pipeline.fireChannelRead 后, 就是 ChannelPipeline 中所需要做的工作了.    
  
OP_WRITE 处理  
OP_WRITE 可写事件代码如下. 这里代码比较简单, 没有详细分析的必要了.  
```
if ((readyOps & SelectionKey.OP_WRITE) != 0) {
    // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
    ch.unsafe().forceFlush();
}
```
  
OP_CONNECT 处理  
最后一个事件是 OP_CONNECT, 即 TCP 连接已建立事件.
```
if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
    // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
    // See https://github.com/netty/netty/issues/924
    int ops = k.interestOps();
    ops &= ~SelectionKey.OP_CONNECT;
    k.interestOps(ops);
    unsafe.finishConnect();
}
```
OP_CONNECT 事件的处理中, 只做了两件事情:  
>1. 正如代码中的注释所言, 我们需要将 OP_CONNECT 从就绪事件集中清除, 不然会一直有 OP_CONNECT 事件.  
>2. 调用 unsafe.finishConnect() 通知上层连接已建立.  
  
unsafe.finishConnect() 调用最后会调用到 pipeline().fireChannelActive(), 产生一个 inbound 事件, 通知 pipeline 中的各个 handler TCP 通道已建立(即 ChannelInboundHandler.channelActive 方法会被调用) 到了这里, 我们整个 NioEventLoop 的 IO 操作部分已经了解完了, 接下来的一节我们要重点分析一下 Netty 的任务队列机制.  

### 源码分析之任务队列机制  
我们已经提到过, 在Netty 中, 一个 NioEventLoop 通常需要肩负起两种任务, 第一个是作为 IO 线程, 处理 IO 操作; 第二个就是作为任务线程, 处理 taskQueue 中的任务. 这一节的重点就是分析一下 NioEventLoop 的任务队列机制的.  
  
#### Task的添加  
普通 Runnable 任务  
NioEventLoop 继承于 SingleThreadEventExecutor, 而 SingleThreadEventExecutor 中有一个 Queue taskQueue 字段, 用于存放添加的 Task. 在 Netty 中, 每个 Task 都使用一个实现了 Runnable 接口的实例来表示. 例如当我们需要将一个 Runnable 添加到 taskQueue 中时, 我们可以进行如下操作:
```
EventLoop eventLoop = channel.eventLoop();
eventLoop.execute(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, Netty!");
    }
});
```
当调用 execute 后, 实际上是调用到了 SingleThreadEventExecutor.execute() 方法, 它的实现如下:
```
@Override
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }
    boolean inEventLoop = inEventLoop();
    if (inEventLoop) {
        addTask(task);
    } else {
        startThread();
        addTask(task);
        if (isShutdown() && removeTask(task)) {
            reject(); 
        }
    }
    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    } 
}
```
!addTaskWakesUp表示“添加任务时，是否唤醒线程”?但是，怎么使⽤取反了。这样反倒变成了，“添加任务时，是否【不】唤醒线程”。具体的原因是为什么呢?
真正的意思是，“添加任务后，任务是否会⾃动导致线程唤醒”。    
对于 Nio 使用的 NioEventLoop ，它的线程执⾏任务是基于 Selector 监听感兴趣的事件，所以当任务添加到 taskQueue 队列中时，线程是⽆感知的，所以需要调用 #wakeup(boolean inEventLoop) ⽅法，进⾏主动的唤醒。  
对于 Oio 使⽤的 ThreadPerChannelEventLoop ，它的线程执行是基于 taskQueue 队列监听( 阻塞拉取 )事件和任务，所以当任务添加到 taskQueue 队列中时，线程是可感知的，相当于说，进⾏被动的唤醒。  
  
添加任务的 addTask 方法的源码如下:
```
protected void addTask(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }
    if (isShutdown()) {
        reject();
    }
    taskQueue.add(task);
}
```
因此实际上, taskQueue 是存放着待执行的任务的队列.  
   
schedule 任务  
除了通过 execute 添加普通的 Runnable 任务外, 我们还可以通过调用 eventLoop.scheduleXXX 之类的方法来添加一个定时任务. EventLoop 中实现任务队列的功能在超类 SingleThreadEventExecutor 实现的, 而 schedule 功 能的实现是在 SingleThreadEventExecutor 的父类, 即 AbstractScheduledEventExecutor 中实现的. 在 AbstractScheduledEventExecutor 中, 有以 scheduledTaskQueue 字段: 
```
Queue<ScheduledFutureTask<?>> scheduledTaskQueue;
```
scheduledTaskQueue 是一个队列(Queue), 其中存放的元素是 ScheduledFutureTask. 而 ScheduledFutureTask 我们很容易猜到, 它是对 Schedule 任务的一个抽象. 我们来看一下 AbstractScheduledEventExecutor 所实现的 schedule 方法:
```
@Override
public  ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    ObjectUtil.checkNotNull(command, "command");
    ObjectUtil.checkNotNull(unit, "unit");
    if (delay < 0) {
        throw new IllegalArgumentException(
                String.format("delay: %d (expected: >= 0)", delay));
    }
    return schedule(new ScheduledFutureTask<Void>(
            this, command, null,
            ScheduledFutureTask.deadlineNanos(unit.toNanos(delay))));
}
```
这是其中一个重载的 schedule, 当一个 Runnable 传递进来后, 会被封装为一个 ScheduledFutureTask 对象, 这个对象会记录下这个 Runnable 在何时运行、已何种频率运行等信息. 当构建了 ScheduledFutureTask 后, 会继续调用另一个重载的 schedule 方法:
```
 <V> ScheduledFuture<V> schedule(final ScheduledFutureTask<V> task) {
    if (inEventLoop()) {
        scheduledTaskQueue().add(task);
    } else {
        execute(new OneTimeTask() {
            @Override
            public void run() {
                scheduledTaskQueue().add(task);
            } 
        });
    }
    return task;
}
```
在这个方法中, ScheduledFutureTask 对象就会被添加到 scheduledTaskQueue 中了.  

#### 任务的执行  
当一个任务被添加到 taskQueue 后, 它是怎么被 EventLoop 执行的呢? 让我们回到 NioEventLoop.run() 方法中, 在这个方法里, 会分别调用 processSelectedKeys() 和 runAllTasks() 方法, 来进行 IO 事件的处理和 task 的处理. processSelectedKeys() 方法我们已经分析过了, 下面我们来看一下 runAllTasks() 中到底有什么名堂吧. runAllTasks 方法有两个重载的方法, 一个是无参数的, 另一个有一个参数的. 首先来看一下无参数的 runAllTasks:
```
protected boolean runAllTasks() {
    fetchFromScheduledTaskQueue();
    Runnable task = pollTask();
    if (task == null) {
        return false;
    }
    for (;;) {
        try {
        task.run();
        } catch (Throwable t) {
           logger.warn("A task raised an exception.", t);
        }
        task = pollTask();
        if (task == null) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
        return true;
        }
    } 
}
```
我们前面已经提到过, EventLoop 可以通过调用 EventLoop.execute 来将一个 Runnable 提交到 taskQueue 中, 也可以通过调用 EventLoop.schedule 来提交一个 schedule 任务到 scheduledTaskQueue 中. 在此方法的一开始调用的 fetchFromScheduledTaskQueue() 其实就是将 scheduledTaskQueue 中已经可以执行的(即定时时间已到的 schedule 任务) 拿出来并添加到 taskQueue 中, 作为可执行的 task 等待被调度执行. 它的源码如下: 
```
private void fetchFromScheduledTaskQueue() {
    if (hasScheduledTasks()) {
        long nanoTime = AbstractScheduledEventExecutor.nanoTime();
        for (;;) {
            Runnable scheduledTask = pollScheduledTask(nanoTime);
            if (scheduledTask == null) {
                break; 
            }
            taskQueue.add(scheduledTask);
        } 
    }
}
```
接下来 runAllTasks() 方法就会不断调用 task = pollTask() 从 taskQueue 中获取一个可执行的 task, 然后调用它 的 run() 方法来运行此 task.
注意 , 因为 EventLoop 既需要执行 IO 操作, 又需要执行 task, 因此我们在调用 EventLoop.execute 方法提交任务时, 不要提交耗时任务, 更不能提交一些会造成阻塞的任务, 不然会导致我们的 IO 线程得不到调度, 影响整个程序的并发量.  
  
#### 整体回顾    
![](https://github.com/YufeizhangRay/image/blob/master/Netty/%E5%BE%AA%E7%8E%AF.jpeg)  
```
1: @Override
2: protected void run() {
3:    for (;;) {
4:        try {
5:            switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
6:                case SelectStrategy.CONTINUE: // 默认实现下，不不存在这个情况。  
7:                      continue;
8:                case SelectStrategy.SELECT:
9:                      // 重置 wakenUp 标记为 false
10:                     // 选择( 查询 )任务
11:                     select(wakenUp.getAndSet(false));
12:
13:                     // 'wakenUp.compareAndSet(false, true)' is always evaluated
14:                     // before calling 'selector.wakeup()' to reduce the wake-up
15:                     // overhead. (Selector.wakeup() is an expensive operation.)
16: 
17:                     // However, there is a race condition in this approach.
18:                     // The race condition is triggered when 'wakenUp' is set to
19:                     // true too early.
20: 
21:                     // 'wakenUp' is set to true too early if:
22:                     // 1) Selector is waken up between 'wakenUp.set(false)' and
23:                     //    'selector.select(...)'. (BAD)
24:                     // 2) Selector is waken up between 'selector.select(...)' and
25:                     //    'if (wakenUp.get()) { ... }'. (OK)
26: 
27:                     // In the first case, 'wakenUp' is set to true and the
28:                     // following 'selector.select(...)' will wake up immediately.
29:                     // Until 'wakenUp' is set to false again in the next round,
30:                     // 'wakenUp.compareAndSet(false, true)' will fail, and therefore
31:                     // any attempt to wake up the Selector will fail, too, causing
32:                     // the following 'selector.select(...)' call to block
33:                     // unnecessarily.
34: 
35:                     // To fix this problem, we wake up the selector again if wakenUp
36:                     // is true immediately after selector.select(...).
37:                     // It is inefficient in that it wakes up the selector for both
38:                     // the first case (BAD - wake-up required) and the second case
39:                     // (OK - no wake-up required).
40:
41:                     // 唤醒。原因，⻅见上⾯面中⽂文注释
42:                     if (wakenUp.get()) {
43:                         selector.wakeup();
44:                     }
45:                     // fall through
46:                 default:
47:             }
48:
49:             // TODO 1007 NioEventLoop cancel ⽅方法
50:             cancelledKeys = 0;
51:             needsToSelectAgain = false;
52:
53:             final int ioRatio = this.ioRatio;
54:             if (ioRatio == 100) {
55:                 try {
56:                     // 处理理 Channel 感兴趣的就绪 IO 事件
57:                     processSelectedKeys();
58:                 } finally {
59:                     // 运⾏行行所有普通任务和定时任务，不不限制时间
60:                     // Ensure we always run tasks.
61:                     runAllTasks();
62:                 }
63:             } else {
64:                 final long ioStartTime = System.nanoTime();
65:                 try {
66:                     // 处理理 Channel 感兴趣的就绪 IO 事件
67:                     processSelectedKeys();
68:                 } finally {
69:                     // 运⾏行行所有普通任务和定时任务，限制时间
70:                     // Ensure we always run tasks.
71:                     final long ioTime = System.nanoTime() - ioStartTime;
72:                     runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
73:                 }
74:             }
75:         } catch (Throwable t) {
76:             handleLoopException(t);
77:         }
78:         // TODO 1006 EventLoop 优雅关闭
79:         // Always handle shutdown even if the loop processing threw an exception.
80:         try {
81:             if (isShuttingDown()) {
82:                 closeAll();
83:                 if (confirmShutdown()) {
84:                     return;
85:                 }
86:             }
87:         } catch (Throwable t) {
88:             handleLoopException(t);
89:         }
90:     }
91: }
```
