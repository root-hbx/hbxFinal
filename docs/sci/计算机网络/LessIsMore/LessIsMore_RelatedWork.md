# Related Work

>To reduce PFC messages, lots of end-to-end congestion control mechanisms [1], [2], [4], [6], [9], [37], [46], [49]–[52] are proposed in recent years. They aim to maintain persistent queue length low, which avoids triggering PFC messages. However, it usually requires 1 RTT for the sources to receive the congestion feedback. Within 1 RTT, it is the switch’s local mechanisms (e.g., buffer management and flow control) that determine whether PFC messages are generated. Thus, DSH is complementary to them. Besides the congestion control mechanisms, lots of studies focus on mitigating the impairments of PFC in other ways. PLB [12] leverages load balancing to alleviate PFC’s head-of-line problem. Several approaches (i.e., TCP-Bolt [6], Tag-ger [7], GFC [8], ITSY [11]) focus on detecting, avoiding, and recovering from PFC deadlocks. To reduce the queue buildup caused by incast traffic, P-PFC [10] uses the derivative of buffer change to predict the buffer occupancy, and proactively generates PFC messages to avoid queue buildup. In compar-ison, DSH focuses on efficient headroom allocation and thus is orthogonal to them.

为了减少PFC消息，近年来提出了许多端到端的拥塞控制机制[1]，[2]，[4]，[6]，[9]，[37]，[46]，[49]–[52]。它们的目标是保持持续的队列长度较低，从而避免触发PFC消息。

然而，通常需要1个往返时间（RTT）让源端接收到拥塞反馈。在1个RTT内，决定是否生成PFC消息的是交换机的本地机制（例如缓冲区管理和流控制）。因此，DSH对它们是互补的。

除了拥塞控制机制外，许多研究还关注以其他方式缓解PFC的问题。**PLB [12]** 利用负载平衡来减轻PFC的头阵线问题。一些方法（例如**TCP-Bolt [6]、Tagger [7]、GFC [8]、ITSY [11]**）专注于检测、避免和从PFC死锁中恢复。为了减少因汇聚流量引起的队列增加，**P-PFC [10]** 使用缓冲区变化的导数来预测缓冲区占用，并主动生成PFC消息以避免队列增加。

相比之下，DSH专注于高效的头部空间分配，因此与它们是正交的。
### 附注：
#### PLB [12] 利用负载平衡来减轻PFC的头阵线问题
https://zhuanlan.zhihu.com/p/587503952
在这种上下文中，PLB 可能指的是 "Packet Level Balancing"，而 PFC 可能指的是 "Priority Flow Control"。这是在数据中心网络或其他网络环境中的一些技术和机制。
1. **PLB (Packet Level Balancing):** Packet Level Balancing 是一种==负载平衡的技术==，它**操作在数据包层面**，确保网络中的==流量被均匀地分配到不同的路径或资源==上，以提高网络性能和避免某些路径或资源过载的问题。
2. **PFC (Priority Flow Control):** Priority Flow Control 是一种在以太网中的流量控制机制，用于处理网络拥塞和确保关键流量的可靠传输。PFC 允许设备在有需要的情况下暂停流量，以避免数据包的丢失。

在您提到的语境中，PLB 可能被用于通过负载平衡来缓解 PFC 的头阵线问题。这可能是指通过动态地调整流量的分布，使得网络中的流量更加均匀，从而减轻 PFC 导致的头阵线问题。

头阵线通常是指在使用 PFC 时，由于某些流量优先级得到了更多的资源，导致其他流量的排队等待时间增加，产生延迟和性能问题。

请注意，这只是基于我对您提供的信息的理解。确切的含义可能需要参考特定文献或上下文。
#### TCP-Bolt [6]
https://en.wikipedia.org/wiki/Bolt_(network_protocol)
#### Tagger [7]
...
#### GFC [8]
...
#### ITSY [11]
...
#### P-PFC [10] 
远程直接内存访问（RDMA）技术迅速改变了当今数据中心应用的格局。对于RDMA网络的拥塞控制是一个关键的挑战。
作为端到端的第3层拥塞控制机制，数据中心QCN（DCQCN）缓解了基于优先级的流量控制（PFC）的不公平性和头阵线阻塞问题。
然而，一个无丢包的网络即使启用了DCQCN也不能保证低延迟。
当网络拥塞发生时，交换机队列仍会由于端到端解决方案的响应延迟而积聚。

在本文中，我们提出了预测性PFC（P-PFC）以降低RDMA网络的尾延迟。P-PFC**监控缓冲区占用的导数**，预测未来PFC触发的发生，并**提前主动触发PFC暂停**。其==好处在于可以保持缓冲区使用在较低水平==，因此**可以控制尾延迟**。初步评估结果表明，在许多场景中，P-PFC可以将尾延迟减少到标准PFC的一半以上，而不损害吞吐量和平均延迟。根据我们的实验，P-PFC还可以保护无辜流。据我们所知，这是首次利用导数改进无丢包RDMA网络中的PFC。