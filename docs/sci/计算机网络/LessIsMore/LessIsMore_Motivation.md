# Motivation

In this section, we present the problem of the current headroom allocation scheme.
### A. Headroom Occupies Considerable Memory

It is expected that most of the memory should serve as shared buffer to absorb bursty traffic without triggering PFC messages. However, with the current buffer allocation scheme, the headroom buffer occupies considerable memory, which can significantly squeeze “footroom” buffer3 and result in frequent PFC messages.

==预期大多数内存应该作为共享缓冲区，以吸收突发流量，而不触发PFC消息!!!!!!==**然而，使用当前的缓冲区分配方案，头部缓冲区占用了相当大的内存，可能会显著挤压“余地”缓冲区**，导致频繁的PFC消息。

Specifically, the current buffer allocation scheme indepen- dently reserves a static headroom for every ingress queue [1], [3]. Assume that each ingress queue requires η headroom. The total headroom size (denoted by h) is given by

具体而言，当前的缓冲区分配方案独立为==每个入口队列==保留了一个==静态的头部缓冲区==[1]，[3]。假设每个入口队列都需要η的头部缓冲区。总头部缓冲区大小（用h表示）由以下公式给出：

> $h=Np × Nq × η$(3)

where $Np$ is the number of ingress ports, $Nq$ is the number of queues per port, and η is given by Eq. (1).

其中，$Np$​是入口端口的数量，$Nq$​是每个端口的队列数量，η由方程（1）给出。

With such method, MMU has to allocate worst-case head- room for every ingress queue, and headroom can occupy a large fraction of memory. For example, Broadcom Trident2 switching chip contains 12MB memory. It has 32 40GbE ports (i.e., Np = 32 and C = 40Gbps). For each port, the PFC standard supports 8 queues (i.e., Nq = 8). Assume that the MTU is 1500B (i.e., LM T U = 1500B) and Dprop = 1.5μs, MMU needs to allocate ∼5.33MB memory for headroom buffer in total, which occupies 44.4% of total memory.

采用这种方法，MMU**必须为每个入口队列分配最坏情况下所需的头部缓冲区，而头部缓冲区可能占用大量内存**。例如，Broadcom Trident2交换芯片包含12MB内存。它有32个40GbE端口（即，Np=32Np​=32，C=40GbpsC=40Gbps）。对于每个端口，PFC标准支持8个队列（即，Nq=8Nq​=8）。假设MTU为1500B（即，LMTU=1500BLMTU=1500B）和Dprop=1.5μsDprop​=1.5μs，MMU需要总共分配约5.33MB内存用于头部缓冲区，占用总内存的44.4％。(性价比太低)

With the growing link capacity, this situation gets worse. The link speed of production DCN has grown from 1Gbps to 40Gbps and 100Gbps in the past decade [4], [13] and is continuously growing. With higher link speed, MMU needs to allocate more headroom to avoid buffer overflow. However, the buffer size is limited by the chip area and cost, and thus cannot scale with the switching capacity [14]–[16]. As a result, the fraction of required headroom becomes increasingly large, significantly squeezing the footroom buffer. Fig. 4 depicts the trend of buffer size and the fraction of required headroom in Broadcom’s switching chips. The switch buffer size per unit of capacity has decreased by 4× in the last decade (from 157μs to 37μs), while the fraction of required headroom has increased by 56% (from 43% to 67%).

随着链路容量的增长，这种情况变得更糟。在过去的十年中，生产数据中心网络的链路速度已经从1Gbps增长到40Gbps和100Gbps[4]，[13]，并且仍在持续增长。随着链路速度的提高，MMU需要分配更多的头部缓冲区以避免缓冲区溢出。然而，由于芯片面积和成本的限制，缓冲区大小无法随着交换容量的增加而扩展[14]–[16]。因此，**所需头部缓冲区的比例变得越来越大，严重挤压了余地缓冲区**。图4描述了Broadcom交换芯片中缓冲区大小和所需头部缓冲区比例的趋势。在过去的十年里，每单位容量的交换机缓冲区大小已经减小了4倍（从157μs减少到37μs），而所需头部缓冲区的比例已增加了56％（从43％增加到67％）。

Without enough “footroom” buffer, PFC messages can be frequently triggered, which may result in serious performance impairments (e.g., head-of-line blocking, congestion spread- ing, collateral damage) and even network deadlocks.

**没有足够的“余地”缓冲区，PFC消息可能会经常触发（见background中的最后一条），这可能导致严重的性能损害**（例如，头部阻塞，拥塞传播，附带损害），甚至网络死锁。

To quantitatively demonstrate the performance degradation brought by inadequate buffer, we perform a large-scale ns-3 simulation on a 256-server leaf-spine topology (more details in §V-B). The congestion control algorithm is PowerTCP [37], which can effectively keep persistent queue length low. We use the widely-used web search workload [27] to generate realistic DCN traffic. The total network load is 90%. Fig. 5 shows the average flow completion time (FCT) with different buffer sizes. The FCT with 14MB buffer is 78.1% worse than that with 30MB buffer.

为了定量地展示由不足够的缓冲区引起的性能降低，我们在一个256服务器叶脊拓扑结构上进行了大规模的ns-3模拟（更多细节见§V-B）。==拥塞控制算法采用PowerTCP== [37]，该算法可以有效地保持持续队列长度较低。我们使用广泛使用的Web搜索工作负载[27]生成真实的数据中心网络流量。总网络负载为90％。图5显示了不同缓冲区大小的平均流完成时间（FCT）。具有14MB缓冲区的FCT比具有30MB缓冲区的FCT差了78.1％。

To alleviate this problem, current network operators have to restrict the number of priority queues [3]. However, this ad hoc approach can aggravate the head-of-line blocking problem, as different services cannot be isolated and the congestion of one point can spread to the entire network. Furthermore, lots of studies [17], [38]–[41] have shown that multiple service queues can greatly improve the network performance. Restrict- ing the number of queues prevents the network applications from benefiting from them.

==为了缓解这个问题，当前的网络运营商不得不限制优先级队列的数量==[3]。然而，这种临时的方法可能会加剧头部阻塞问题，因为无法隔离不同服务，一个点的拥塞可能传播到整个网络。**此外，许多研究[17]，[38]–[41]表明，多个服务队列可以极大地提高网络性能。限制队列的数量会阻止网络应用程序从中受益。**
### B. Current Headroom Allocation Scheme is Inefficient

Despite the increasingly large fraction of headroom, the current static and independent headroom allocation scheme (referred to as SIH) is still quite inefficient and wasteful. This is due to three reasons:

随着头部缓冲区的占用比例越来越大，当前的静态且独立的头部分配方案（称为SIH）相当低效和浪费。这是由于三个原因：

(1) Not all queues need to occupy headroom. An ingress queue needs to occupy headroom only when it is congested and its queue length is higher than the Xoff threshold. In reality, it is unlikely that all queues are congested at the same time [42]. Despite this, SIH conservatively allocates worst- case headroom size for all ingress queues. As a result, most headroom buffer keeps unused.

(2) Traffic heading for different ingress queues at the same port shares the capacity of uplink. In a port, all ingress queues are connected to the same uplink and thus the traffic heading for them shares the link capacity. When a traffic class of a port needs to be paused, its traffic arriving rate should be lower than C as long as other traffic classes also have in-flight packets on the uplink. Thus, the actual required headroom size can be lower than η.

(3) The upstream device is not always sending traffic at full rate. When allocating headroom buffer for an ingress queue, SIH assumes that the upstream device will always send packets at line rate before PAUSE frame takes effect. However, the upstream queue can be empty. As a result, the sending rate of upstream device can be lower than link capacity and the headroom can be over-allocated.

(1) **并非所有队列都需要占用头部缓冲区**。**只有在拥塞时，入口队列才需要占用头部缓冲区**，并且其队列长度高于Xoff阈值。**实际上，不太可能所有队列同时拥塞**[42]。尽管如此，SIH为所有入口队列保守地分配最坏情况下的头部缓冲区大小。因此，大多数头部缓冲区保持未使用状态。

(2) 在**同一端口前往不同入口队列的流量共享上行链路的容量**。在一个端口中，所有入口队列都连接到相同的上行链路，因此前往它们的流量共享链路容量。==当端口的某个流量类别需要暂停时，只要其他流量类别在链路上仍有在途数据包，其到达速率应低于C==。因此，theoretically！实际所需的头部缓冲区大小可以低于η。【review：$η = 2 (C · Dprop + L[MTU] ) + 3840B$】

(3) **上游设备并非总是以满速发送流量**。在为入口队列分配头部缓冲区时，SIH假设上游设备在PAUSE帧生效之前始终以线速发送数据包。==然而，上游队列可以为空。因此，上游设备的发送速率可能低于链路容量，头部缓冲区可能被过度分配。==

To quantitatively demonstrate the inefficiency, we perform a large-scale ns-3 simulation with the same settings as the previous one except that the congestion control algorithm is DCQCN [1], which has higher buffer occupancy. To examine the efficiency of headroom, we extract the local maximum values of headroom occupancy, which indicates the actual required headroom size. Fig. 6 shows the distribution of headroom utilization for a port at the local maximum point. The headroom utilization is only 4.96% at the median and 25.33% at the 99th percentile.

为了定量地展示这种低效性，我们进行了一次大规模的ns-3模拟，设置与前一模拟相同，只是==拥塞控制算法为DCQCN ==[1]，其具有更高的缓冲区占用率。为了检查头部的效率，我们提取了头部占用的局部最大值，这表示实际所需的头部缓冲区大小。图6显示了一个端口在局部最大点的头部利用率分布。在中位数处，头部利用率仅为4.96％，而在第99百分位处为25.33％。

Note that the inefficiency of SIH is inherent and cannot be avoided by simply adjusting the headroom buffer size. This is because SIH has to allocate headroom buffer in the worst case to ensure that buffer never overflows, even though the worst case rarely occurs. Thus, to efficiently utilize the headroom buffer, we need to seek another headroom allocation scheme.

请注意，**SIH的低效性是固有的，不能通过简单调整头部缓冲区大小来避免。这是因为SIH必须在最坏情况下分配头部缓冲区，以确保缓冲区永远不会溢出，即使最坏情况很少发生**。因此，为了有效利用头部缓冲区，我们需要寻找另一种头部分配方案。
#### ques1: 拥塞控制算法采用PowerTCP
https://blog.csdn.net/qq_45577173/article/details/131988302
#### ques2: 拥塞控制算法为DCQCN
###### 1. 简介：
DCQCN（Distributed Congestion Control for Quantum Network）是一种专为高性能计算和数据中心网络设计的拥塞控制算法。它主要用于以太网网络，特别是用于数据中心网络中的高速、低延迟网络。

以下是DCQCN的一些关键特点和原则：
1. **分布式控制：** DCQCN是一种**分布式拥塞控制算法**，意味着网络中的多个节点协同工作以检测和缓解拥塞。==每个节点都能够监测网络的状态并采取相应的措施==来减轻拥塞。
2. **Quantum Flow Control：** DCQCN使用Quantum Flow Control的概念，通过==监测队列的深度和延迟等信息==，动态调整发送速率，以避免拥塞的发生。这种方法有助于确保网络的高性能和低延迟。
3. **速率限制：** DCQCN通过在网络中的节点之间交换信息，实施==速率限制==，以确保每个节点都以适当的速率发送数据，以防止网络拥塞。
4. **自适应性：** DCQCN是自适应的，能够根据网络状况的变化==调整自身的参数和行为==。这使得它能够灵活地适应不同负载和网络条件。
5. **基于ECN的反馈：** DCQCN==使用显式拥塞通知（ECN）来获取有关网络状态的反馈==。当网络中的节点检测到拥塞时，它们可以通过ECN向其他节点发送信号，从而触发相应的拥塞控制机制。

请注意，DCQCN通常与数据中心网络中的RoCE（RDMA over Converged Ethernet）等技术一起使用，以提供高性能、低延迟的数据中心通信。这种拥塞控制算法的详细实现和参数可能会根据具体的网络设备和配置而有所不同。如果你对具体的实现或配置有疑问，建议查阅相关文档或与网络设备的制造商联系。
###### 2. 实现原理：
以下是DCQCN的一般实现原理：
1. **ECN（Explicit Congestion Notification）：** DCQCN使用ECN来获取网络状态的反馈。==ECN是一种网络拥塞指示机制==，==允许网络中的设备向其他设备发送拥塞通知，而不必通过丢包来进行隐式拥塞检测==。DCQCN的节点通过检测队列深度和延迟等信息来判断网络拥塞，并使用ECN来通知其他节点。
2. **Quantum Flow Control：** DCQCN采用Quantum Flow Control的思想，这是一种动态调整发送速率的机制。通过监测网络中的拥塞状态，DCQCN可以调整流的速率，以确保网络性能的最优化。
3. **分布式控制：** DCQCN是一种分布式拥塞控制算法，意味着网络中的多个节点共同参与拥塞控制。每个节点都能够独立地监测和响应拥塞情况，以最大程度地减少网络拥塞对性能的影响。
4. **速率限制：** DCQCN通过速率限制来确保每个节点都以适当的速率发送数据。这可以防止某一节点过于快速地向网络发送数据，从而导致拥塞。
5. **自适应性：** DCQCN是自适应的，可以根据网络的动态变化进行调整。这使得它能够适应不同负载和网络条件，保持网络性能的稳定性和高效性。
6. **RoCE（RDMA over Converged Ethernet）：** DCQCN通常与RoCE等技术一起使用，以提供高性能、低延迟的数据中心通信。RoCE是一种支持在以太网上实现RDMA（Remote Direct Memory Access）的协议，它与DCQCN协同工作以提供优越的数据中心网络性能。
需要注意的是，DCQCN的具体实现可能会因制造商、设备和配置而有所不同。如果你对具体的实现或配置有疑问，建议查阅相关文档或与网络设备的制造商联系。
###### 3. DCQCN如何做到：允许网络中的设备向其他设备发送拥塞通知
DCQCN（Distributed Congestion Control for Quantum Network）是一种拥塞控制算法，用于以太网网络中的高性能计算和数据中心网络。

以下是DCQCN节点如何通过检测队列深度和延迟等信息来判断网络拥塞，并使用ECN来通知其他节点的基本原理：
1. **队列深度检测：** DCQCN节点会**监测其发送队列和接收队列的深度**。队列深度是指队列中等待传输的数据包数量。如果队列深度==超过一定的阈值==，可能表明网络出现拥塞。这是因为拥塞通常会导致队列中积累大量未传输的数据包。
2. **延迟检测：** DCQCN还会监测网络中的传输延迟。延迟是数据从发送端到接收端所需的时间。在拥塞情况下，由于网络资源的争用，传输延迟可能会增加。节点可以通过==比较当前延迟与预设的阈值来判断==是否存在拥塞。
3. **ECN（Explicit Congestion Notification）：** 如果节点检测到拥塞的迹象，它可以通过ECN来通知其他节点。**ECN是一种在IP数据包头部中的标志位，用于明确⚠️指示网络中的拥塞**。当节点==检测到拥塞时，它可以设置数据包的ECN标志位，然后发送到网络==。
4. **ECN的传递：** 被设置了ECN标志位的数据包在传输过程中会被传递到网络的其他节点。中间路由器和交换机可以检测到这个标志位，并在传递给下一个节点时继续保持。这样，拥塞信息可以通过数据包传递到整个网络。
5. **其他节点的响应：** *接收到带有设置了ECN标志位的数据包的其他节点* 可以**根据自身的拥塞控制策略来做出相应的响应**。这可能包括减小发送速率、调整窗口大小等措施，以缓解网络拥塞。

总体而言，DCQCN利用队列深度、延迟等信息来判断网络拥塞，通过设置ECN标志位来明确通知其他节点。这种分布式的拥塞控制机制有助于协调网络中各个节点，以确保高性能和低延迟的数据传输。请注意，具体实现和参数可能会因硬件、协议版本和网络配置而有所不同。