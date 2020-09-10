## 项目效果展示
项目展示分为项目一（简易RPC框架实现）和项目二（研究生创新能力评价系统）展示，主要展示项目中个人责任内容，展示形式通过截图形式表现。
* [1.简易RPC框架实现](#1)
 ** [1.1项目介绍](#1.1)
 ** [1.1项目技术](#1.2)
 ** [1.1项目设计](#1.3)
 ** [1.1RPC服务端启动](#1.4)
 ** [1.1RPC客户端启动及服务调用](#1.5)
 ** [1.1服务变更通知](#1.6)
<h2 id="1">简易RPC框架实现</h2>
<h3 id="1.1">项目介绍</h3>
项目实现了简单的 RPC 框架，其中主要包括四个模块，分别为 rpc-core 模块、rpc-provider 模块、rpc-consumer 模块、rpc-interfaces 模块。
- rpc-core 模块提供了 RPC 框架的核心实现，其中包括，简单的负载均衡实现、远程通讯实现、配置 相关的实现、动态代理的实现、序列化的实现。
- rpc-provider 模块实现 RPC 框架服务端的启动，rpc-consumer 模块实现 RPC 框架客户端的启动。
- rpc-interfaces 模块提供了 RPC 服务。

<h3 id="1.2">项目技术</h3>
Spirng+Netty+ZooKeeper+Protostuff+cglib 

<h3 id="1.3">项目设计</h3>
- 使用 Spring IoC，将对象之间的相互依赖关系交给 IoC 容器管理，并由 IoC 容器完成对象的注入。
- 使用 Spring XML schema，实现了用 xml 配置来使用 RPC 框架。
- 使用 Netty 作为客户端和服务器的通信实现方案，使用主从 Reactor 多线程模型将客户端连接请求
和业务处理分别由不同线程池完成，通过实现 Netty 预置编解码器，解决粘包问题。
- ZooKeeper 作为服务的注册中心，实现服务的暴露和引用。
- 通过 ZooKeeper 的 Watcher 机制，实现客户端对服务变更的监听。
- 使用 cglib 作为动态代理实现方案，log4j 作为打印和输出方案，Protostuff 作为序列化实现方案。

<h3 id="1.4">RPC服务端启动</h3>
![RPC服务端启动](../images/RPC服务端启动.jpg)
<h3 id="1.5">RPC客户端启动及服务调用</h3>
![RPC客户端启动及服务调用](../images/RPC客户端启动及服务调用.jpg)
<h3 id="1.6">服务变更通知</h3>
![服务变更通知](../images/服务变更通知.jpg)
### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Origin-9/oriNote.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
