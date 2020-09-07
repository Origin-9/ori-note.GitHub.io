#### Netty是什么？

Netty是一款提供**异步的**、**事件驱动**，用于**构建高性能网络应用程序**的框架。

Netty是基于NIO，并且参考Reactor设计模式来进行设计的。

极大简化并优化了TCP和UDP套接字服务器等网络编程，稳定性，安全性也更好

支持多种协议，也可以自己定义协议

#### 为什么要用Netty

- 相比较JDK中提供的AIO，决解了其中的bug，性能有了提高，其实现的ByteBuf会使得更低的资源消耗，内存复制

- 简单强大的线程模型
- 简单的API，提供阻塞和非阻塞模型
- 自带编解码器解决TCP拆包粘包问题
- 安全性不错，有对SSL/TLS的支持
- 社区活跃
- 成熟稳定，很多开源框架使用，例如：Dubbo，Elasticsearch

#### Reactor模型

##### Reactor单线程模型

单线程模型就是只指定一个线程执行客户端连接和读写操作，也就是在一个Reactor中完成，

将NioEventLoopGroup线程数设置为1

##### Reactor多线程模型

一个单Reactor线程中进行客户端连接处理，然后业务处理交给线程池

##### 主从Reactor线程模型

多个Reactor线程（bossGroup线程池）处理客户端连接，业务处理交给线程池（workerGroup）

#### 核心组件

##### BootStrap/ServerBootStrap

均继承自AbstractBootstrap

引导就是配置应用程序的过程,使应用程序代码在后台可以连接和启动所有的组件。

1. `Bootstrap` 通常使用 `connet()` 方法连接到远程的主机和端口，作为一个 Netty TCP 协议通信中的客户端。另外，`Bootstrap` 也可以通过 `bind()` 方法绑定本地的一个端口，作为 UDP 协议通信中的一端。
2. `ServerBootstrap`通常使用 `bind()` 方法绑定本地的端口上，然后等待客户端的连接。
3. `Bootstrap` 只需要配置一个线程组— `EventLoopGroup` ,而 `ServerBootstrap`需要配置两个线程组— `EventLoopGroup` ，一个用于接收连接，一个用于具体IO事件的处理。

##### EventLoopGroup/EventLoop

一个EventLoopGroup包含一个或多个EventLoop

`EventLoop` 负责处理注册到其上的`Channel` 处理 I/O 操作

> 扩展了JUC的SecheduledExecutorService

##### Channel

通道 Channel 接口是Netty对网络操作的抽象。聚合了一组功能，包括IO功能，连接功能，也包含了Netty相关的功能，包括获取Channel关联的EventLoop，获取缓冲分配器ByteBufAllocator和pipeline等

##### channelFuture

Netty 是异步非阻塞的，所有的 I/O 操作都为异步的。

因此，我们不能立刻得到操作是否执行成功，但是，你可以通过 `ChannelFuture` 接口的 `addListener()` 方法注册一个 `ChannelFutureListener`，当操作执行成功或者失败时，监听就会自动触发返回结果。

##### channelHandler和channelPipeline

`ChannelHandler` 是消息的具体处理器。他负责处理读写操作、客户端连接等事情

`ChannelPipeline` 为 `ChannelHandler` 的链，提供了一个容器并定义了用于沿着链传播入站和出站事件流的 API 。当 `Channel` 被创建时，它会被自动地分配到它专属的 `ChannelPipeline`。

我们可以在 `ChannelPipeline` 上通过 `addLast()` 方法添加一个或者多个`ChannelHandler` ，因为一个数据或者事件可能会被多个 Handler 处理。当一个 `ChannelHandler` 处理完之后就将数据交给下一个 `ChannelHandler` 。

##### Bytebuf

netty在ByteBuffer的基础上实现的新的，功能功能更加强大的ByteBuf

优点：

- 容量动态*扩容*
- 读写采用不同的指针

- 可以用户自定义缓冲区类型
- 通过内置复合缓冲区实现零拷贝
- 支持方法链式调用
- 支持引用计数
- 支持池化

> 发送数据时，传统的实现方式的拷贝情况：
>
> 1. 从磁盘读取到内核read buffer
> 2. 从内核缓冲区拷贝到用户缓冲区
> 3. 从用户缓冲区拷贝到内核的socket buffer
> 4. 从内核socket buffer 拷贝到网卡接口
>
> 通过 java 的FileChannel.transferTo，避免上述的2，3步骤
>
> 零拷贝：操作系统层面指避免在用户态和核心态之间来回拷贝数据，netty层面指对数据操作的优化
>
> 1. 提供的 CompositeBuf 类，将多个ByteBuf逻辑上合并成一个，避免拷贝
> 2. Bytebuf提供slice操作，将ByteBuf分解成共享同一存储区域的Bytebuf
> 3. 通过FileChannel.transferTo，直接将文件缓冲区的数据发送到目标 `Channel`

###### 使用模式

###### -堆缓冲区

数据存放在JVM的堆空间，通过将数据存储在数组中实现

![image-20200818113057857](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200818113057857.png)

###### -直接缓冲区

直接缓冲区属于堆外分配的直接内存，避免了数据从内部缓冲区拷贝到直接缓冲区的过程

不在堆中，处理前要先复制到数组中

![image-20200818113122116](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200818113122116.png)

###### -复合缓冲区

提供一个或多个ByteBuf的组合视图，可以根据需要添加和删除不同类型的ByteBuf

如果要访问，需要先将内容拷贝到堆内存中，再进行访问

![image-20200818113255083](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200818113255083.png)

###### ByteBufHoleder接口

通过子类实现ByteBufHolder接口，可以添加在自己需要的数据结构，实现自定义的数据缓冲区

###### ByteBuf分配

###### -非池化的

Unpooled工具类来创建未池化的ByteBufAllocator

###### -池化的

netty默认使用PooledByteBufAlloactor

###### ByteBufUtil提供了操作ByteBuf的静态方法

###### 引用计数

当ByteBuf引用计数为0时，系统会回收缓冲区

##### BytetoMessageDecoder/MessagetoByteEncoder

netty为字节到消息或者消息到字节的解码器，编码器实现提供了基类，通过继承可以自己实现编解码器

> TCP粘包问题？
>
> TCP是面向字节流的，他会拼接和拆分应用层协议的数据
>
> 没有明确的消息边界，就会导致数据接收方无法拼接接收的数据，导致数据出现问题

netty有自带的编解码器解决TCP粘包问题

基于换行符的`LineBasedFrameDecoder`

基于分隔符的`DelimiterBasedFrameDecoder`

基于长度的`FixedLengthFrameDecoder`

也可以自己实现：例如自定义协议，数据长度+数据

###### 编解码器

继承ByteToMessageCoder，实现decoder，encoder方法

CombinedChannelDuplexHandler类可以将编码器和解码器组合起来

#### 启动过程

##### 服务端

创建服务器启动引导ServerBootStrap示例

配置Reactor线程池[NioEventLoopGroup]（bossGroup，workerGroup）

.channel指定IO模型

通过.channelHandler() 给引导类创建channelInitializer，并且指定服务端消息的业务处理逻辑

调用引导类的bind方法，绑定端口，等待客户端连接

##### 客户端

创建BootStrap引导类

配置线程组NioEventLoopGroup

.channel指定IO模型

通过.channelHandler() 给引导类创建channelInitializer，并且指定客户端消息的业务处理逻辑

调用引导类connect连接服务端

#### 长连接，心跳机制

长连接说的就是 client 向 server 双方建立连接之后，即使 client 与 server 完成一次读写，它们之间的连接并不会主动关闭，后续的读写操作会继续使用这个连接。长连接的可以省去较多的 TCP 建立和关闭的操作，节约时间。对于频繁请求资源的客户来说，非常适用长连接。

在 client 与 server 之间在一定时间内没有数据交互时, 即处于 idle 状态时, 客户端或服务器就会发送一个特殊的数据包给对方, 当接收方收到这个数据报文后, 也立即发送一个特殊的数据报文, 回应发送方, 此即一个 PING-PONG 交互。所以, 当某一端收到心跳消息后, 就知道了对方仍然在线, 这就确保 TCP 连接的有效性