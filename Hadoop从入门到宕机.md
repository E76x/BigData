# Hadoop

# 1.介绍&概览

Hadoop是一个开源的分布式计算框架，用于**存储和处理大规模数据集**。

Hadoop 框架主要由四个模块组成，这四个模块协同运行以形成 Hadoop 生态系统：

1. Hadoop Distributed File System (HDFS)：作为 Hadoop 生态系统的主要组件，HDFS 是一个分布式文件系统，可提供对应用数据的高吞吐量访问，而无需预先定义架构。
2. Yet Another Resource Negotiator (YARN)：YARN 是一个资源管理平台，负责管理集群中的计算资源并使用它们来调度用户的应用。它在整个 Hadoop 系统上执行调度和资源分配。
3. MapReduce：MapReduce 是一个用于大规模数据处理的编程模型。通过使用分布式和并行计算算法，MapReduce 可以沿用处理逻辑，并帮助编写将大型数据集转换为可管理数据集的应用。
4. Hadoop Common：Hadoop Common 包括其他Hadoop 模块使用和共享的库和实用程序。

# 2.特点

- scalability扩容能力强
- economical 成本低
- efficiency 效率高
- reliability 可靠性强

# 3.分布式存储系统核心属性

**无限扩展支撑海量数据存储:**

- **特点：** 分布式存储系统能够无缝地扩展以支持大规模数据存储需求。系统能够轻松地添加新的节点，从而提供对海量数据的持续支持，而不会遭遇性能瓶颈。

**元数据记录:**

- **特点：** 分布式存储系统记录关于存储的元数据信息，这包括文件的位置、大小、权限等信息。这样的元数据记录使得系统能够快速定位文件位置，实现高效的文件查找和访问。

**分块存储:**

- **特点：** 存储系统将大文件划分成小块，并分布存储在多个节点上。这种分块存储的方式允许对数据进行并行操作，从而提高系统的读写效率。同时，也使得系统能够更好地应对大规模数据的存储和处理需求。

  (HDFS中的文件在物理上是分块存储的default block = 128M,参数位于hdfs-default.xml中的dfs.blocksize)

**副本机制:**

- **特点：** 为了保障数据的安全性和容错性，分布式存储系统通常采用副本机制。系统会将数据的多个副本存储在不同的节点上，确保即使发生节点故障，数据仍然可用。冗余的存储机制提高了系统的可靠性和容错性。



# 4.HDFS集群

HDFS提供对大文件的高效管理，**满足"写一读多"的需求**。一旦文件写入，不再需要修改。

适用场景包括大文件、数据流式访问、一次写入多次读取、低成本部署（使用廉价PC）、高容错性。

## NameNode

**元数据管理：** NameNod**e负责管理HDFS中所有文件和目录的元数据**，包括文件的命名空间、文件和目录的层次结构、权限信息以及每个文件的数据块的位置等。

**单点故障：** Hadoop的NameNode是HDFS的单点故障（Single Point of Failure，SPOF）。**这意味着如果NameNode发生故障，整个HDFS文件系统将不可用**。为了提高可用性，Hadoop引入了Secondary NameNode来定期合并和检查文件系统的编辑日志。

**持久化存储：** NameNode的元数据是持久化存储的，通常存储在本地磁盘上。这种设计使得NameNode能够快速加载文件系统的元数据，并提供快速的文件系统命名空间操作。

**编辑日志和镜像：** NameNode维护一个编辑日志（Edit Log），记录文件系统的变更操作。同时，还会定期创建一个镜像文件（FsImage），用于持久化文件系统的快照。这两者结合使用，以便在NameNode启动时快速恢复文件系统状态。

**块映射表：** NameNode维护一个块映射表，记录每个文件的数据块的位置信息。这有助于客户端在读取文件时快速定位所需的数据块。

**通信：** NameNode与HDFS客户端、DataNode以及其他Hadoop组件进行通信，以执行文件系统操作，如创建、删除、重命名文件等。

## DataNode

DataNode是Hadoop分布式文件系统（HDFS）的另一个关键组件，负责存储实际的数据块。每个DataNode节点都负责存储一部分HDFS中的数据块。

**数据块存储：** 当客户端向HDFS写入文件时，文件被划分为数据块，这些数据块被分布存储在不同的DataNode上。DataNode负责存储这些数据块，并在需要时向客户端提供这些数据。

**心跳和块报告：** DataNode会定期向NameNode发送心跳消息，通知其自己的存活状态。此外，DataNode还发送块报告，其中包含它所管理的数据块的列表。

## **Secondary NameNode:**

Secondary NameNode并不是NameNode的备份，而是用于协助NameNode的辅助节点。其主要任务是定期合并和压缩NameNode的编辑日志（Edit Log），以避免其过度增长。

**编辑日志合并：** Secondary NameNode定期从NameNode获取编辑日志，将其合并成一个镜像（FsImage）文件，并将压缩后的编辑日志返回给NameNode。这有助于快速恢复NameNode的状态，特别是在发生故障时。

**非故障容错：** Secondary NameNode并不是NameNode的热备份，它并不能在NameNode故障时立即接管其职责。因此，在Hadoop 2.x版本后，Hadoop社区提出了High Availability（HA）架构，通过引入多个NameNode实例来提高系统的可用性，而不再依赖Secondary NameNode。

## HDFS读数据步骤

![HDFS读数据步骤](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126203221191.png)

1. Client向NameNode发起RPC请求，确定文件block位置。
2. NameNode返回block列表和每个block的DataNode地址。
3. DataNode地址按照集群拓扑结构排序，考虑网络距离和心跳机制的状态。
4. Client选取排序靠前的DataNode读取block，若为本地DataNode，则直接获取数据。
5. 通过建立Socket Stream（FSDataInputStream），循环调用read方法读取数据块。
6. 每读取一个block进行checksum验证，出现错误时通知NameNode并从其他DataNode备份读取。
7. 最终合并所有block形成完整文件。

## 	HDFS写数据步骤

![HDFS写数据步骤](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126203141182.png)

1. Client通过RPC请求向NameNode上传文件，NameNode检查文件和权限信息，若通过则创建文件记录，否则返回异常信息。
2. Client请求NameNode确定第一个block的DataNode地址，根据备份策略选择可用的DataNode（例如 A、B、C）。
3. Client向选定的DataNode A 发起上传请求，建立pipeline，依次调用 B、C，并逐级返回给 Client。
4. Client开始向 A 上传第一个block，以64K的packet为单位，A传递给 B，B传递给 C，每传一个packet放入应答队列等待应答。
5. 若某DataNode在写数据时宕机，系统会执行透明的恢复步骤，确保数据不丢失，并完成未写完的block的备份。
6. 数据通过pipeline传输，逐个发送ack，最终由第一个DataNode A 将ack发送给Client。
7. 当一个block传输完成后，Client再次请求NameNode上传下一个block。

# 5.MapReduce

MapReduce是一种用于处理和生成大规模数据集的编程模型和处理引擎。

思想核心是“先分再合，分而治之”。

MapReduce的核心思想分为两个阶段：Map阶段和Reduce阶段。

![MapReduce流程图](D:\Study\MyStudy\大数据\开发\Hadoop\MapReduce流程图.png)

## Map阶段

在MapTask执行时，输入数据来源于HDFS的Block。默认情况下，每个Split对应一个Block。在WordCount例子中，假设Map的输入数据是字符串“aaa”。

1. **切分输入数据：** 输入数据被切分成若干个小块，每个小块称为一个Input Split。
2. **Mapper任务执行：** 用户自定义的Mapper函数被并行应用于每个Input Split。Mapper将每个输入记录转换为一系列中间键值对（key-value pair）。
3. **中间键值对输出：** Mapper输出的中间键值对包含了经过映射处理后的数据。
4. **Partitioner决定ReduceTask：** 这些键值对被分区（Partition）并发送到Reducer。
5. **内存缓冲区：** 数据经过分区方法标记分区，然后发送到环形缓冲区，默认大小100MB。
6. **Spill（溢写）：** 当缓冲区达到80%时，进行排序和溢写。排序按照key的索引进行字典顺序排序，使用快排。
7. **Merge（合并）：** 最终，MapTask完成时，所有溢写文件将被归并成一个文件。即使Map的输出很小，也至少会有一个溢写文件存在。

## Reduce阶段

1. **Copy阶段：** 每个Reduce拉取对应分区的数据，先存储到内存，内存不足时存储到磁盘。
2. **Merge阶段：** 拉取所有数据后，采用归并排序将内存和磁盘中的数据排序和合并。
3. **Reducer输出文件：** 对数据调用reduce方法，经过不断Merge后，生成一个“最终文件”。

## ReShuffle机制

**Map方法之后Reduce方法之前这段处理过程叫Shuffle。**

**Map阶段：**

1. 数据经过分区方法标记分区，然后发送到环形缓冲区，默认大小100MB。
2. 当缓冲区达到80%时，进行排序和溢写。排序按照key的索引进行字典顺序排序，使用快排。
3. 溢写产生溢写文件，需进行归并排序，可以应用Combiner操作。

**Reduce阶段：**

1. 每个Reduce拉取对应分区的数据，先存储到内存，内存不足时存储到磁盘。
2. 拉取所有数据后，采用归并排序将内存和磁盘中的数据排序和合并。
3. 在进入Reduce方法前，可以对数据进行分组操作。

4. 按照分区将数据存储到磁盘，等待Reduce端拉取。

# 6.YARN集群

Hadoop YARN（Yet Another Resource Negotiator）是Hadoop的一个集群资源管理器，用于有效地管理和调度集群中的资源

![YARN官方架构图](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126202646271.png)

## 特点和主要组件

1. **分离资源管理与计算调度：** YARN将资源管理（Resource Manager）和作业调度（Application Master）分离，使得Hadoop集群可以同时运行多个应用程序。
2. **Resource Manager：** 负责整个集群的资源管理，分配和监控。它接收来自各个Node Manager的资源报告，并根据应用程序的需求动态分配资源。
3. **Node Manager：** 运行在集群中的每个节点上，负责监控本地资源使用情况，并向Resource Manager报告可用资源。
4. **Application Master：** 用于管理特定应用程序的资源请求、分配和监控。每个应用程序都有一个独立的Application Master。
5. **Container：** YARN使用容器的概念，将资源划分为容器，并动态分配给不同应用程序的任务。

## 工作流程

![image-20240126203029565](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126203029565.png)

1. 用户通过客户端向YARN的ResourceManager提交应用程序，ResourceManager为应用程序分配第一个容器，并要求NodeManager启动ApplicationMaster。
2. 一旦ApplicationMaster启动成功，它向ResourceManager注册，允许用户通过ResourceManager查看应用程序运行状态。
3. ApplicationMaster为程序内的任务向ResourceManager申请资源，并监控任务状态。一旦资源分配成功，ApplicationMaster与对应的NodeManager通信，要求启动任务。
4. NodeManager设置任务运行环境，将任务启动命令写入脚本，并通过运行脚本启动任务。
5. 各个任务通过RPC向ApplicationMaster汇报状态和进度，使其随时了解任务运行情况。在应用程序运行期间，用户可通过RPC向ApplicationMaster查询当前运行状态。
6. 应用程序完成后，ApplicationMaster注销并关闭自身。

## 调度器策略

### FIFO Scheduler（先进先出调度器）

**特点：**

- 简单而直观的调度器，按照作业提交的先后顺序依次分配资源。
- 不考虑作业的资源需求和优先级。

**适用场景：**

- 适用于小型集群或者对调度性能要求不高的场景。
- 任务的优先级不会变高，因此高优先级的作业需要等待。不适合需要灵活性、资源隔离和更复杂调度需求的大规模集群。

### Capacity Scheduler（容量调度器）

**特点：**

- 多队列调度器，每个队列有独立的资源配额。
- 可以为不同的用户或应用程序分配独立的资源。
- 在一个队列内部，资源的调度是采用的是先进先出(FIFO)策略。

**优势：**

- 更灵活，支持队列的层次结构，能够更精细地控制资源分配。
- 在运行时可以动态调整队列的资源配额。

![image-20240126203953669](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126203953669.png)

### Fair Scheduler（公平调度器）

**特点：**

- 公平调度器试图在所有应用程序之间平均分配资源，避免某个应用长时间占用全部资源。
- 不同应用程序之间的资源分配更为公正。

**适用场景：**

- 适用于需要公平分享集群资源的场景，防止某个应用长时间占用全部资源。

**特殊特性：**

- 支持动态资源池调整和作业优先级，更注重公平性。
- 通过权重和最小份额来实现对资源的公平分配。

#### 如何理解公平共享

- 有两个用户A和B，每个用户都有自己的队列。
- A启动一个作业，由于没有B的需求，它分配了集群所有可用的资源。
- 然后B在A的作业仍在运行时启动了一个作业，经过一段时间，A,B各自作业都使用了一半的资源。
- 现在，如果B用户在其他作业仍在运行时开始第二个作业，它将与B的另一个作业共享其资源，因此B的每个作业将
- 拥有资源的四分之一，而A的继续将拥有一半的资源。

- 结果是资源在用户之间公平地共享。



![image-20240126204206927](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240126204206927.png)

