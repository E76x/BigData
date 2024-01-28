# ZooKeeper

![ZK框架](https://cdn.processon.com/userId2-65acc16767d2da5ab7357f53?e=1705824120&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:akflgRdDpNClp_CFdSPIiu7OPkQ=?s=100_100)

# 1.定义

ZooKeeper 是一个分布式的协调服务,通常用于协助分布式系统中的各个部分进行协同工作。提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

## 集群角色

- leader

   处理所有的事务请求（写请求），可以处理读请求，集群中只能有一个leader。

- follower

​     只能处理读请求，同时作为leader的候选节点，即如果leader宕机，follower节点要参与到新的leader选举中，有可能成为新的leader节点。

- observer

​     只能处理读请求，不能参与选举。

# 2.特点

- Zookeeper:一个领导者(Leader)，多个跟随者(Follower）组成的集群。
- Zookeepe集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以zookeeper适合安装奇数台服务器。
- 全局数据一致:每个server保存一份相同的数据副本，client无论连接到哪个Server，数据都是一致的。
- 更新请求顺序执行，来自同一个client的更新请求按其发送顺序依次执行，即先进先出。
- 数据更新原子性，一次数据更新要么成功，要么失败。
- 实时性，在一定时间范围内，client能读到最新数据。

# 3.提供的服务

- 统一命名服务：在分布式环境下，对应用/服务进行统一命名，方便识别。
- 统一配置管理：在集群中确保所有节点的配置信息一致，通过ZooKeeper管理配置信息，通过客户端监听znode实现同步。
- 统一集群管理：ZooKeeper实现实时监控节点状态变化，被广泛应用于NameNode的HA、HBase的HA以及Flink等场景。
- 服务器节点动态上下线：ZooKeeper可用于监控和处理服务器节点的动态上下线。
- 软负载均衡：ZooKeeper可以协助实现软负载均衡，确保分布式系统中的负载分布均匀。(在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新客户端请求)

**分布式锁服务：**

- 写锁（排它锁）：通过ZooKeeper上的znode实现，创建znode表示获取锁，删除znode表示释放锁。
- 读锁（共享锁）：在预先存在的znode（如/distribute_lock）下创建临时顺序编号目录节点，编号最小的获得锁。

**队列管理：**

- 同步队列：创建临时目录节点，在约定目录下监听节点数目是否达到要求。
- 先进先出队列：创建临时顺序编号目录节点，出队按编号顺序进行。

# 4.工作机制

- **基于观察者模式设计：** ZooKeeper确实使用了观察者模式，其中客户端可以注册Watcher，用于监听ZooKeeper上的节点状态变化。一旦某个节点的状态发生变化，ZooKeeper会通知所有注册了Watcher的客户端，使得它们能够及时做出相应的反应。
- **存储和管理关键数据：** ZooKeeper负责存储和管理分布式系统中关键的共享数据，这些数据通常用于协调和同步分布式系统的各个部分。ZooKeeper的数据模型类似于文件系统，其中的每个ZNode都可以存储少量的数据。
- **通知机制：** 通过观察者模式，ZooKeeper实现了一种通知机制，使得分布式系统中的各个节点能够感知到关键数据的变化。这种通知机制对于实现分布式锁、配置管理等场景非常有用。
- **ZooKeeper = 文件系统 + 通知机制：** 这个比喻是合适的。ZooKeeper的数据模型和操作方式类似于文件系统，但它不仅仅是一个简单的存储系统，还提供了强大的通知机制，使得分布式系统能够更加灵活地响应数据的变化。

> ZooKeeper基于观察者模式设计，允许客户端注册观察者。当数据状态变化时，ZooKeeper通知已注册的观察者做出相应反应。同时，它存储和管理关键数据，可以把ZooKeeper视为文件系统加上通知机制。

# 5.监听机制

![zk监听机制](https://cdn.processon.com/userId2-65b46648ea71e54329792445?e=1706325081&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:92Zl-GJKfyi7zRH6iH8MWguK4F8=?s=100_100)

- 客户端注册监听它关心的znode，当znode发生变化（数据改变、节点删除、子目录节点增加删除）
  时，ZooKeeper 会通知客户端。监听机制保证 ZooKeeper 保存的数据发生改变能快速的响应到监听
  了该节点的应用程序。
- 监听器的工作机制，其实是在客户端会专门创建一个监听线程，在本机的一个端口上等待
  ZooKeeper集群发送过来事件。

- 监听工作原理：ZooKeeper 的 Watcher 机制主要包括客户端线程、客户端 WatcherManager、
  Zookeeper 服务器三部分。客户端在向 ZooKeeper 服务器注册的同时，会将 Watcher 对象存储在客
  户端的 WatcherManager 当中。当 ZooKeeper 服务器触发Watcher 事件后，会向客户端发送通知，客户
  端线程从 WatcherManager 中取出对应的 Watcher 对象来执行回调逻辑。

## Watcher特性

- **一次性触发：**
  - 一个 Watch 事件是一个一次性的触发器，即客户端只会收到一次这样的信息。
- **异步的：**
  - ZooKeeper 服务器发送 Watcher 的通知事件到客户端是异步的。不能期望监控到节点的每次变化，ZooKeeper 只能保证最终的一致性，而无法保证强一致性。
- **轻量级：**
  - Watcher 通知非常简单，只是通知发生了事件，而不会传递事件对象的具体内容。
- **客户端串行：**
  - 执行客户端 Watcher 回调的过程是一个串行同步的过程。

# 6.ZK的核心功能

Zookeeper 提供了三个核心功能：文件系统、通知机制和集群管理机制。

## 文件系统

Zookeeper 存储数据的结构，类似于一个文件系统。每个节点称之为znode，买个znode都是类似于K-V的结构，每个节点的名字相当于key，每个节点中都保存了对应的数据，类似于key-value中的value。

## 通知机制

当某个 client 监听某个节点时，当该节点发生变化时，zookeeper就会通知监听该节点的客户端，后续根据客户端的处理逻辑进行处理。

## 集群管理机制

zookeeper 本身是一个集群结构，有一个 leader 节点，负责写请求，多个 follower 节点负责相应读请求。

并且在 leader 节点故障的时候，会根据选举机制从剩下的 follower 中选举出新的leader。

# 7.ZK的数据结构

ZooKeeper数据模型的结构整体上可以看作是一棵树，每个节点称做一个znode。每一个znode默认能够存储1MB的数据，每个znode都可以通过其路径唯一标识。

![ZK的数据结构](https://cdn.processon.com/userId2-65acc4c8498763371ee751ff?e=1705824984&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:cC8gGpjr8EDVmyV_SJCLdCTQA8Q=?s=100_100)

### 节点分类

- **持久节点（PERSISTENT）：**
  - 一旦创建成功不会被删除，除非客户端主动发起删除请求。
- **持久顺序节点（PERSISTENT_SEQUENTIAL）：**
  - 会在用户路径后面拼接一个不会重复的自增数字后缀，一旦创建成功不会被删除，除非客户端主动发起请求。
- **临时节点（EPHEMERAL）：**
  - 当创建该节点的客户端断开连接后就会被自动删除。
- **临时顺序节点（EPHEMERAL_SEQUENTIAL）：**
  - 创建时会在用户路径后面拼接一个不会重复的自增数字后缀，当创建该节点的客户端断开连接后就会被自动删除。
- **容器节点（CONTAINER）：**
  - 一旦子节点被删除完，该节点就会被服务端自动删除。
- **带过期时间的持久节点（PERSISTENT_WITH_TTL）：**
  - 带有超时时间的节点，如果超时时间内没有子节点被创建，就会被删除。需要开启服务端配置 `extendedTypesEnabled=true`。
- **带过期时间的持久顺序节点（PERSISTENT_SEQUENTIAL_WITH_TTL）：**
  - 创建节点时会在用户路径后面拼接一个不会重复的自增数字后缀，带有超时时间，如果超时时间内没有子节点被创建，就会被删除。需要开启服务端配置 `extendedTypesEnabled=true`。

# 8.选举机制

当zookeeper集群中出现如下两种情况之一就会进行leader的选举：

- 服务器初始化（及第一次选举）
- 服务器运行期间无法与Leader保持联系(第二次选举)

## 第一次启动选举

![第一次启动选举](https://img2020.cnblogs.com/blog/2191745/202111/2191745-20211117220324388-519408543.png)

假设有五台服务器组成的 ZooKeeper 集群，它们的 ID 从1到5，**都是最新启动的且没有历史数据**。让我们按顺序来看它们启动时可能发生的情况：

1. 服务器1启动，处于LOOKING状态，因为此时只有它一台服务器。
2. 服务器2启动，与服务器1进行通信,**两者都没有历史数据，所以id值较大的服务器2胜出**，但由于没有达到超过半数以上的服务器同意选举它(在这个例子中为 >= 3)，所以服务器1、2仍然保持LOOKING状态。
3. 服务器3启动，成为Leader，因为有三台服务器选举了它，超过了半数以上的同意。
4. 服务器4启动，由于前面已经有半数以上的服务器选举了服务器3，所以它成为Follower。
5. 服务器5启动，同样成为Follower。

## 非第一次启动选举

> 原来选出的leader挂掉，出现障碍，需要重新选举时。
>
> 三个概念：
>
> SID:服务器ID。用来唯一标识一台ZooKeeper集群中的机器，每台机器不能重复，和myid一致。
>
> ZXID:事物ID。ZXID是一个事物ID,用来标识一次服务器状态的变更。在某一时刻，集群中的每台机器的ZXID值不一定完全一致，这和ZooKeeper服务器对于客户端“更新请求”’的处理逻辑有关。
>
> Epoch:每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加。

**无法联系 Leader 的情况：**

- 如果 server5 无法与当前 Leader 保持联系，它可能认为当前的 Leader 宕机。

**Leader 的重新选举：**

- server5 会发起 Leader 的重新选举，希望成为新的 Leader。

**两种可能的情况：**

- **情况一：集群中已存在 Leader**
  - 当 server5 尝试进行 Leader 选举时，会与其他服务器进行投票和信息交换。
  - 如果集群中已经存在 Leader，server5 会被告知当前集群已经有 Leader 了。
  - 在这种情况下，server5 只需与现有的 Leader 建立联系并同步状态。
- **情况二：集群中不存在 Leader**
  - 假设 ZooKeeper 集群由5台服务器组成，SID分别为1、2、3、4、5，ZXID为8、8、8、7、7，此时 SID 为3 的机器为 Leader。
  - 在某一时刻，server3 与 server5 都宕机，开始 Leader 的重新选举。
  - 选举规则包括：
  - a. 比较 epoch 的值，谁的值大谁胜出；
  - b. 如果 epoch 相同，则比较 ZXID，谁的值大谁胜出；
  - c. 如果 epoch 和 ZXID 也相同，则比较 SID，谁的值大谁胜出。
  - 例如，server1、server2、server4 的 epoch、ZXID、SID 分别为（1，8，1）、（1，8，2）、（1，7，4），则最终的选举结果为 server2 当选为新的 Leader。

## Zookeeper集群为什么最好奇数台?

ZooKeeper集群在宕掉几个ZooKeeper服务器之后，如果剩下的ZooKeeper服务器个数大于宕掉的个数的话整个ZooKeeper才依然可用。

假如我们的集群中有n台ZooKeeper服务器，那么也就是剩下的服务数必须大于n/2。

假如我们有3台，那么最大允许宕掉1台ZooKeeper服务器，如果我们有4台的的时候也同样只允许宕掉1台。

假如我们有5台，那么最大允许宕掉2台ZooKeeper服务器，如果我们有6台的的时候也同样只允许宕掉2台。

综上所述，我们发现2n和2n-1的容忍度是一样的，都是n-1。所以，何必增加那一个不必要的ZooKeeper服务器呢?

# 9. ZAB协议内容
## 什么是ZAB协议?

- ZAB(ZooKeeper Atomic Broadcast原子广播)协议是为分布式协调服务ZooKeeper框架专门设计的一种支持崩溃恢复的原子广播协议。

- 在**ZooKeeper中，主要依赖ZAB协议来实现分布式数据一致性**，基于ZAB协议，ZooKeeper实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性。

- ZAB协议包括两种基本的模式：消息广播、崩溃恢复。

  ## 消息广播

  消息广播概念：

  - 一旦超过半数的Follower服务器与Leader服务器同步，整个集群进入消息广播模式。新加入的服务器会自动进入数据恢复模式，与Leader进行数据同步，然后参与消息广播。

> ZooKeeper写数据的机制是客户端将写请求发送到leader节点，leader节点将数据通过提案请求发送给所有节点，节点写入本地磁盘后发送ACK给leader。Leader收到过半节点的ACK确认(commit)后，发送提交消息给所有节点，节点将消息放入内存(放内存是为了保证高性能)，使其对用户可见。

![zk写数据](https://ask.qcloudimg.com/http-save/yehe-5640963/2e05823fa5c8261a8e7823f439272546.png)

## ZK如何保证数据一致性？

Zookeeper要想保证数据一致性，就需要考虑如下两个情况：

**1.**leader执行提交(commit)了，还没来得及给follower发送提交(commit)的时候，leader宕机了，这个时候如何保证消息的一致性?

**ZAB 的崩溃恢复机制**

- 当leader宕机后，ZooKeeper会选举新的leader。新的leader启动后会检查磁盘上是否有未提交的消息，如果有，会检查其他follower是否已经对该消息进行了提交。如果过半节点已经ACK确认但未提交，新leader将完成提交操作。

**2.**客户端把消息写到leader了，但是leader还没发送提案(proposal)消息给其他节点，这个时候leader宕机了，**leader宕机后恢复**的时候，此消息又该如何处理?

**ZAB 的恢复中删除数据机制**

- 客户端向leader发送消息，但leader未发送提案消息给其他节点前宕机，导致消息写入失败。当leader恢复但成为follower后，发现未提交的消息，这个时候当前的节点就会根据高32位知道目前leader已经切换过了，所以就把当前的消息给删除掉，然后从新的leader中进行同步数据，这样就保证了数据的一致性。

> 消息是有编号的，由高32位和低32位组成，高32位是用来体现是否发生过leader切换的，低32位就是展示消息的顺序的。

# 10.Paxos算法

## 1.介绍

- Paxos算法，是一种基于消息传递且具有高度容错性的一致性算法。
- Paxos算法解决的问题：就是如何在一个分布式系统中对某个数据值达成一致，并且保证不论发生任何异常(机器宕机、网络异常)，都不会破坏整个系统的一致性。

Paxos算法的描述：

- 在一个Paxos系统中，首先将所有节点划分为提议者(Proposers)、接收者(Acceptors)和学习者(Learners)，每个节点都可以身兼数职；

  Proposer（提议者，用来发出提案proposal）
  Acceptor（接受者，可以接受或拒绝提案）
  Learner（学习者，学习被选定的提案，当提案被超过半数的Acceptor接受后为被批
  准）

- 一个完整的Paxos算法流程分为3个阶段：

![paxos算法](https://img-blog.csdnimg.cn/7dc96bea56454610ac8867b145cf4a21.png)

**Prepare阶段：**

- 提议者(Proposers)向多个接收者(Acceptors)发出提议(Propose)，请求承诺(Promise)；

- 接收者(Acceptors)针对收到的提议(Propose)请求，进行承诺(Promise)；

**Accept阶段：**

- 提议者(Proposers)收到多数接收者(Acceptors)承诺的提议(Propose)后，正式向接收者(Acceptors)发出提议(Propose)；

- 接收者(Acceptors)针对收到的提议(Propose)请求，进行接收(Accept)处理；

**Learn阶段：**

- 提议者(Proposers)将形成的决议发送给所有的学习者(Learners)；

## 2.Paxos算法缺点

当系统中有一个以上的提议者(Proposers)时，多个提议者(Proposers)之间相互争夺接收者(Acceptors)，造成迟迟无法达成一致的情况，针对此情况，

一种改进的Paxos算法被提出：从系统中选出一个节点作为Leader，只有Leader能够发起提议。

这样，一次Paxos流程中，只有一个提议者，不会出现活锁的情况；

# 11. CAP原理

CAP原理是指分布式系统中的三个基本属性：一致性（Consistency）、可用性（Availability）、分区容错性（Partition Tolerance）。根据CAP原理，分布式系统中无法同时满足这三个属性，只能在其中选择两个进行权衡。

- **一致性（Consistency）**：所有节点在同一时间具有相同的数据视图，即对系统进行读取操作后，所有的节点都会返回最新写入的数据或者错误。换句话说，一致性要求分布式系统在数据更新后能够立即反映到所有节点上，确保任何时刻数据的一致性状态。
- **可用性（Availability）**：系统提供的服务必须一直处于可用状态，即使出现了部分节点失败或无法响应，仍然需要保证对外提供服务的能力。可用性要求系统能够在有限的时间内对请求做出响应，并且不会因为某些节点的故障而导致整个系统不可用。
- **分区容错性（Partition Tolerance）**：系统能够容忍网络分区（即分布式系统中的节点之间的通信出现故障或延迟），并且系统能够继续正常运行。分区容错性要求系统能够在网络分区后继续对外提供服务，并且保持一致性和可用性。

根据CAP原理，分布式系统只能满足其中的两个属性，而无法同时满足三个。在实际设计和部署分布式系统时，需要根据具体的应用场景和需求来权衡选择。



> 参考:
>
> https://houbb.github.io/2022/05/10/interview-08-zookeeper
>
> https://cloud.tencent.com/developer/article/1927599
>
> https://chinalhr.github.io/post/distributed-systems-consensus-algorithm-paxos/

