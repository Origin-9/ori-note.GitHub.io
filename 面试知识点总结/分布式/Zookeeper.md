### 什么是Zookeeper？

Zookeeper为我们提供了高可用、高性能、稳定的分布式数据一致性解决方案，通常用于诸如数据发布/订阅，分布式锁，命名服务，注册中心等

> Zookeeper数据保存在内存中，性能是非常棒的，再”读“多于写的应用程序中，尤其高性能。

#### 特点：

- 顺序一致性：客户端发起的事务，会严格的按照顺序应用到Zookeeper中
- 原子性：事务的处理结果在集群节点中是一致的
- 单一系统映像：集群节点中数据模型是一致的
- 可靠性：事务被应用，就会持久化

#### 应用场景：

##### 分布式锁

1. 创建锁目录 /lock
2. 当客户端要获取锁时，在 /lock 下创建临时且有序的子节点
3. 客户端获取 / lock 下的节点列表。如果，自己的节点序号是否是列表中最小的，如果是，就获取锁，不是，就监听前一个子节点，获取节点变更通知后，重新执行本步骤
4. 执行业务代码，完成后，删除节点

##### 命名服务

可以通过Zookeeper的顺序节点生成全局唯一的ID

##### 数据发布/订阅

将数据发布在Zookeeper被监听的节点上，通过Watcher机制，其他客户端监听该节点，来获取数据的变更通知

### Zookeeper重要概念

#### 数据模型

采用层次化的多叉树结构，根节点用"/"来代表，每个数据节点被称为znode，每个znode有唯一的路径标识。

> Zookeeper主要用来协调服务，不是用来存储业务数据的，每个节点数据大小最大1M。

#### znoed类型

- 持久节点：一旦创建一直存在（即使Zookeeper宕机），直到删除
- 临时节点：临时节点生命周期和客户会话绑定，会话消失节点消失。**临时节点只能做叶子节点，不能创建子节点**

- 持久顺序节点：除了具有持久结点的特性之外，子节点的名称还具有顺序性。
- 临时顺序节点：除了具备临时结点的特性之外，子节点的名称还具有顺序性。

##### znode数据结构

- stat：状态信息
- data：数据的具体内容

stat部分状态信息：

cZxid：创建时的事务id

mZxid：该节点最后一次更新时事务id

cversion：子节点版本号

dataVersion：数据节点内容版本号

aclVersion：节点的ACL版本号

numChildren：子结点个数

#### ACL(权限控制)

对znode操作的权限：

CREATE，READ，WRITE，DELETE，ADMIN（能设置节点ACL的权限）

对于身份认证，提供以下几种方式：

world：默认方式，任何用户都可以访问

auth：不适用任何id，代表任何已经认证的用户

digest：用户名：密码认证方式

ip：对指定ip限制

#### Watcher（事件监听器）

Zookeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，会将事件通知到感兴趣的客户端上去。

#### 会话（Session）

Session可以看作是Zookeeper服务器与客户端之间的一个TCP长连接，通过长连接，可以发送请求，接收响应，接收Watcher事件通知，用心跳检测保持连接的有效

Session一属性：sessionTimeout，代表会话超时时间，在sessionTimeout规定时间内能够连接上服务器，那么会话继续有效

创建会话之前，服务器会给客户端分配一个sessionID，这个是Zookeeper会话的一个重要标识，全局唯一。

### Zookeeper集群

**最典型集群模式： Master/Slave 模式**

通常 Master 服务器作为主服务器提供写服务，其他的 Slave 服务器通过异步复制的方式获取 Master 服务器最新的数据提供读服务

> 集群数量最好是奇数台，例如三台机器，一台宕机集群照常运行，四台机器也只能允许一台宕机，容忍度是一样。

#### 角色

- Leader：为客户端提供读和写的服务，负责发起投票和决议，更新系统状态。

- Follower：为客户端提供读服务，如果是写服务则转发给 Leader。在选举过程中参与投票。
- Observer：为客户端提供读服务器，如果是写服务则转发给 Leader。不参与选举过程中的投票，也不参与“过半写成功”策略。

#### 服务器状态

- **Locking**：寻找或者选举Leader
- **LEADING** ：Leader 状态，对应的节点为 Leader。
- **FOLLOWING** ：Follower 状态，对应的节点为 Follower。
- **OBSERVING** ：Observer 状态，对应节点为 Observer，该节点不参与 Leader 选举。

### ZAB

在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，ZooKeeper 实现了一种主备模式中各个服务节点之间的数据一致性。

ZAB状态：

- ELECTION：选主阶段
- DISCOVERY：连接上leader
- SYNCHRONIZATION：同步
- BROADCAST：过渡到广播状态，开始对外服务

**ZAB有两种模式：恢复模式和广播模式**

>**为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务**。

#### 选举

选举时机：集群启动/leader宕机、断电、网络延迟高，follower心跳检测监测到后，进入选举模式

##### 选票数据结构

- logicClock：表示该服务器发起的第几轮投票
- state：当前服务器的状态
- self_id：当前服务器的myid
- self_zxid：当前服务器上保存的数据的最大zxid
- vote_id：被推举的服务器myid
- void_zxid：被推举的服务器所保存的数据最大zxid

##### 投票流程

1. 自增选举轮次

   每个服务器开启新一轮投票都会把logicClock自增

2. 初始化投票

   服务器在广播自己的选票时，会将自己的投票箱清空，该投票箱记录所受到的投票。

3. 发送初始化选票

   每个服务器最开始都会将票投给自己然后广播出去

4. 接收外部投票

   服务器尝试获取其他服务器的投票。无法获取就确认自己和其他服务器的有效连接，有效再次发送自己的投票；否则进行连接

5. 判断收到投票的选举轮次

   - 外部logicClock大，重置投票箱，将自己的logicClock更新，对比收到的该投票，看是否需要变更的投票，然后投出去

   - 外部logicClock小，忽略
   - 相等，选票PK

6. 选票PK

   - 收到的logicClock大，将自己的变更
   - 若一致，比较两者vote_zxid，若收到的大，则将自己的vote_zxid和vote_myid更新为收到的然后广播出去，把收到的投票和自己的放入投票箱
   - 若vote_zxid一致，则比较vote_myid，若收到的大，则将自己的vote_zxid和vote_myid更新，并广播出去，把收到的投票和自己的投票放入投票箱

7. 统计选票

   若有确定过半服务器认可了自己的投票，则终止投票

8. 更新服务器状态

#### 同步

1. leader等待follower连接
2. follower连接后会将最大的zxid发送给leader
3. leader根据follower的zxid确定同步点
4. 完成同步后通知follower已经成为uptodata状态了
5. follower接受到uptodata后可以接受client连接请求了

##### 同步种类

leader会从内存数据库中提取出提议缓存队列，初始化peerLastZxid，minCommittedLog，maxCommitedLog

###### 直接差异化同步

peerastZxid介于min和max之间

Leader会向Leaner发送DIFF指令，通知leaner进行同步，然后发送PROPOSAL数据包和COMMIT指令数据包

###### 先回滚在差异化同步

前任leader处理新事务且，写入事务日志当中，恰好要将该proposal广播出去的时候，宕机

集群进行了新的选举，该服务重新连接

此时leader发现learner存在自己没有的事务记录，则进行回滚，回滚到leader上存在的，最接近peerLastZxid的zxid上，然后进行直接差异化同步

###### 仅回滚同步

peerLastZxid大于maxCommittedLog，leader会要求learner回滚到ZXID值为maxCommitedLog对应的事务操作

###### 全量同步

peerLastZxid 小于minCommittedLog

#### 广播（正常情况下，主从复制）

leader会将客户端request转化为事务proposal，分配zxid，广播给learner，（leader和learner之间有一队列）learner将事务写入事务日志之后，给leader返回ack反馈，当接收到半数以上ack则向全部learner发送commit指令