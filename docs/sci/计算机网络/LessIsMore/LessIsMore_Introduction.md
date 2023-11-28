# Introduction

>Lossless network is increasingly attractive in datacenters as it can provide ultra-low latency for applications [1]–[5]. In commodity Ethernet, lossless transmission is achieved by the hop-by-hop Priority-based Flow Control (PFC) mechanism. To avoid packet dropping, a PFC-enabled switch sends a PAUSE frame to its upstream device when its buffer is about to over- flow. Receiving the PAUSE frame, the upstream device holds back the packet transmission. To prevent buffer overflow, a fraction of buffer should be reserved as headroom to absorb in- flight packets before the PAUSE frame takes effect. However, in large-scale networks, PFC messages can result in serious performance issues, such as head-of-line blocking, congestion spreading, collateral damage, and even deadlocks [3], [4], [6]–[12]. Therefore, it is a common belief that PFC should be triggered as few as possible. Ideally, PFC should only serve as a backup method to ensure lossless packet transmissions.
However, due to the recent industrial trends, it is more and more likely that datacenter networks (DCNs) suffer from frequent PFC messages. Specifically, the link speed of DCN has rapidly grown from 1Gbps to 40Gbps/100Gbps and will continuously increase to 400Gbps in the near future [3], [13]. The amount of required headroom is increasingly large as it is positively related to the link speed. Unfortunately, the memorysize in the switching chip cannot keep pace with the link capacity [14]–[16]. This is because datacenter switches usually employ on-chip memory for high-speed and low-latency mem- ory access, and the memory size is limited by the chip area and cost. Specifically, the buffer size (related to switching capacity) has decreased by 4× over the past decade (§III-A). With these trends, a considerable amount of buffer should be reserved as headroom, significantly squeezing the buffer space for accommodating normal traffic. As a result, the queue length can easily hit the PFC pause threshold and PFC messages will be frequently generated. Although recent advances in end-to-end congestion control (CC) algorithms [1]–[4] can keep persistent buffer occupancy low, they cannot completely tackle this issue. This is because end-to-end CC takes at least 1 round-trip time (RTT) to react to traffic changes, and has no control over short-term congestion events, which are very common in DCNs. Specifically, studies have shown that most flows will be finished within 1 RTT in future DCNs [17], [18] and most congestion events will be caused by sub-RTT traffic bursts [19]. Within 1 RTT, it is the buffer management scheme that determines whether PFC messages can be avoided.
In face of the considerable headroom requirement, we find that the current headroom allocation scheme (which is called SIH in this paper) is quite inefficient in headroom utilization. Our experiments show that 75% of headroom keeps unused 99% of the time even when the network load is up to 90% (§III-B). This is because SIH reserves a static amount of worst- case headroom independently for every ingress queue in every port, which wastes a substantial amount of memory due to the following two reasons:
(1) Not all ingress queues need to occupy the headroom. An ingress queue takes up the headroom only when it becomes congested and it is very unlikely that all ports/queues are con- gested at the same time. Thus in most cases, most headroom keeps unused and is wasted.
(2) Every ingress queue is allocated with the worst-case headroom, while the worst case rarely occurs for all ingress queues simultaneously. The worst-case headroom requirement assumes that traffic arrives at the highest rate. In reality, queues of the same ingress port share the common uplink capacity, and thus it is impossible that all ingress queues are receiving traffic at line rate.
Note that SIH’s inefficiency is inherent, lying in its queue- independent and static headroom allocation method. Thus, the problem cannot be simply addressed by adjusting the amount of reserved headroom, which may lead to unacceptable packet loss for lossless networks. In this case, a brand-new headroom allocation scheme is needed to radically resolve this issue.
In light of this, we propose Dynamic and Shared Headroom allocation scheme (DSH) (§IV) to significantly reduce the headroom allocation without risking packet loss. To this end, DSH combines port-level flow control and queue-level flow control. The port-level flow control performs flow control operations on port-wise, which enables DSH to guarantee lossless transmission with a small amount of reserved head- room. It is based on the observation that different ingress queues in the same port naturally share the common uplink capacity. Thus, to avoid packet overflow, there is no need to independently reserve headroom for each ingress queue. Instead, DSH only needs to reserve headroom for each port and make the ingress queues in the same port share the reserved headroom. However, the headroom reduction comes at a cost. Occupying the headroom reserved for each port will pause the entire traffic from the upstream link, which is harmful to performance as performance isolation is violated. Thus, DSH also utilizes queue-level flow control most of the time and makes port-level flow control merely an insurance method against packet loss.
The queue-level flow control performs flow control actions on queue-wise, which is the same as PFC. It is based on the observation that not all queues need to occupy headroom at the same time. Thus, DSH dynamically allocates headroom to each queue only when the queue gets congested. In this way, the headroom is allocated only when truly needed, and thus will not be wasted. Furthermore, DSH enables the headroom to be shared among different ingress queues. In this way, DSH can take advantage of statistical multiplexing to efficiently utilize the allocated headroom.
We evaluate DSH with extensive ns-3 simulations (§V). Our microbenchmarks show that DSH can absorb over 4× more bursty traffic without triggering PFC messages, and can effectively mitigate the impairments (including collateral damage and deadlock) induced by PFC (§V-A). Our large- scale simulations show that DSH reduces the flow completion time by up to ∼57.7% for short fan-in flows and up to ∼31.1% for other flows (§V-B).
The rest of the paper is organized as follows. §II introduces the background of PFC and switch buffer. §III discusses the problem of the current headroom allocation scheme. Next, §IV describes the design of DSH. In §V, we present the evaluations of DSH. Finally, §VII concludes the paper.

无损网络在数据中心中越来越有吸引力，因为它可以为应用程序提供超低延迟[1]-[5]。 在商用以太网中，无损传输是通过逐跳基于优先级的流量控制（PFC）机制来实现的。 
为了避免丢包，*启用 PFC 的交换机* 会在缓冲区即将溢出时向其上游设备发送 PAUSE 帧。上游设备收到PAUSE帧后，暂停报文发送。 
为了防止缓冲区溢出，应保留一小部分缓冲区作为余量，以在暂停帧生效之前吸收正在传输的数据包。 
然而，在大规模网络中，PFC 消息可能会导致严重的性能问题，例如队头阻塞、拥塞扩散、附带损害，甚至死锁 [3]、[4]、[6]–[12] 。 
因此，人们普遍认为应尽可能少地触发 PFC。 理想情况下，PFC 应该仅作为确保无损数据包传输的备份方法。
然而，由于最近的行业趋势，数据中心网络（DCN）越来越有可能受到频繁的PFC消息的影响。 
具体来说，DCN的链路速度已从1Gbps快速增长到40Gbps/100Gbps，并且在不久的将来将继续提高到400Gbps[3]、[13]。 所需的净空量(headroom)越来越大，因为它与链路速度正相关。 
不幸的是，记忆交换芯片的尺寸无法跟上链路容量[14]-[16]。 这是因为数据中心交换机通常采用片上存储器来实现高速、低延迟的存储器访问，而存储器大小受到芯片面积和成本的限制。
具体来说，缓冲区大小（与开关容量相关）在过去十年中减少了 4 倍 (§III-A)。 考虑到这些趋势，需要预留相当大的缓冲区作为净空(headroom)，从而大大挤压了容纳正常流量的缓冲区空间。

因此，队列长度很容易达到PFC暂停阈值，并且会频繁生成PFC消息。 尽管==端到端拥塞控制（CC）算法==[1]-[4]的最新进展可以保持较低的持久缓冲区占用率，但它们不能完全解决这个问题。 这是因为端到端 CC 至少需要 1 个往返时间 (RTT) 才能对流量变化做出反应，并且无法控制 DCN 中常见的短期拥塞事件。 具体来说，研究表明，在未来的DCN中，大多数流量将在1个RTT内完成[17]、[18]，并且==大多数拥塞事件将由亚RTT流量突发引起[19]。 在1个RTT内，缓冲区管理方案决定是否可以避免PFC消息==。
面对相当大的余量需求，我们发现==当前的余量分配方案（本文称为SIH）==在余量利用率方面相当低效。 我们的实验表明，即使网络负载高达 90%，仍有 75% 的净空(headroom)在 99% 的时间内保持未使用状态 (§III-B)。 ==这是因为 SIH 为每个端口中的每个入口队列独立保留静态量的最坏情况余量，由于以下两个原因，这浪费了大量内存：==
(1) 并非所有入口队列都需要占用净空(headroom)。 入口队列仅在变得拥塞时才占用净空，并且所有端口/队列同时拥塞的可能性很小。 因此，在大多数情况下，大部分净空都未使用并被浪费。【*这时候headroom是按照端口内的队列逐一分配的*】
(2) 每个入口队列都分配有最坏情况的余量，而最坏情况很少会同时出现在所有入口队列中。 最坏情况的净空要求假设流量以最高速率到达。 实际上，同一入口端口的队列共享共同的上行链路容量，因此不可能所有入口队列都以线速接收流量。

请注意，SIH 的低效率是==固有的，在于其与队列无关的静态余量分配方法==。 因此，这个问题*不能简单地通过调整保留余量来解决，这可能会导致无损网络无法接受的数据包丢失*。 在这种情况下，就需要一种全新的净空分配方案来从根本上解决这个问题。
鉴于此，我们提出动态和共享余量分配方案（DSH）（§IV），以显着减少余量分配，而不会有丢包的风险。 
为此，DSH结合了端口级流控和队列级流控。 ==端口级流控按端口进行流控操作，使得DSH能够在保留少量余量的情况下保证无损传输==。

它基于以下观察：同一端口中的不同入口队列自然地共享共同的上行链路容量。 因此，为了避免数据包溢出，不需要为每个入口队列独立保留净空。 相反，**DSH 只需要为每个端口预留余量，并使同一端口中的入口队列共享预留的余量**。 【比喻一下：端口是一个静态大池子，每个队列是瓢，可以动态改变自身的储水量】。
然而，==净空的减少是有代价的。 占用为每个端口保留的余量将暂停来自上游链路的整个流量，这对性能有害，因为违反了性能隔离==。 因此，==DSH在大多数情况下也利用队列级流控制，并使端口级流控制仅仅作为防止数据包丢失的保险方法。【point！】==

队列级流控按队列进行流控动作，与PFC相同。 这是基于观察，并非所有队列都需要同时占用净空。 因此，**仅当队列发生拥塞时，DSH 才会为每个队列动态分配空间。 这样，只有在真正需要时才分配净空，不会造成浪费**。 此外，**DSH 允许在不同的入口队列之间共享净空**。 通过这种方式，DSH 可以**利用统计复用来有效地利用分配的余量**。

我们通过广泛的 ns-3 模拟来评估 DSH (§V)。 我们的微基准测试表明，DSH 可以在不触发 PFC 消息的情况下吸收超过 4 倍的突发流量，并且可以有效减轻 PFC 引起的损害（包括附带损害和死锁）（§V-A）。 我们的大规模模拟表明，对于短扇入流，DSH 可以将流完成时间减少高达 ∼57.7%，对于其他流，则可以减少高达 ∼31.1% (§V-B

本文的其余部分安排如下。 §II介绍PFC和开关缓冲器的背景。 §III讨论了当前净空分配方案的问题。 接下来，第 IV 节描述了 DSH 的设计。 在§V 中，我们提出了 DSH 的评估。 最后，第七节总结了本文。

----------
#### 1. Although recent advances in end-to-end congestion control (CC) algorithms [1]–[4] can keep persistent buffer occupancy low, they cannot completely tackle this issue. This is because end-to-end CC takes at least 1 round-trip time (RTT) to react to traffic changes, and has no control over short-term congestion events, which are very common in DCNs.

###### 1. 简要分析：
这段文字讨论了近期在端到端拥塞控制算法方面的进展，指出**尽管这些算法能够保持持久性缓冲区占用率较低，但它们并不能完全解决这个问题**。这是因为端到端拥塞控制==至少需要1个往返时延（RTT）来对流量变化做出反应==，并且对于*短期拥塞事件（在数据中心网络中非常常见）*，端到端拥塞控制无法进行有效的控制。

具体解读如下：
1. **端到端拥塞控制算法的进展：** 提到了最近在端到端拥塞控制算法方面的进展，这些算法旨在**确保网络中持续的缓冲区占用率较低**。这是为了**提高网络性能和减少拥塞**导致的问题。
2. **无法完全解决问题：** 尽管这些算法在维持持久性缓冲区占用率方面表现良好，但它们并不能完全解决问题。这里所指的问题可能涉及到网络拥塞和流量管理的挑战。
3. **端到端拥塞控制的限制：** 提到端到端拥塞控制的限制，其中==最主要的限制是它需要至少1个往返时延（RTT）来对流量变化做出反应==。在==数据中心网络中，这种反应时间可能会导致在短时间内发生的拥塞事件无法得到有效控制==。
4. **短期拥塞事件的普遍性：** 强调了在数据中心网络中短期拥塞事件的普遍性。这意味着网络中可能会发生一些短暂的拥塞，而端到端拥塞控制算法无法在短时间内做出有效反应。
综合来看，文本指出了端到端拥塞控制算法的一些限制，尤其是对于短期拥塞事件的处理不足，这在数据中心网络中是比较常见的情况。
###### 2. end-to-end CC Algorithm：
"端到端拥塞控制算法"（end-to-end congestion control algorithms）是一类用于管理网络拥塞的算法。这些算法的目标是确保网络中的流量在高负载情况下仍然能够有效传输，并防止因拥塞而导致的数据包丢失和网络性能下降。以下是这些算法的一些主要特征：
1. **流量监测：** 端到端拥塞控制算法通常会**监测网络中的流量，以便了解当前网络的状态**。这可能涉及到测量数据包传输的速率、延迟等。
2. **拥塞信号：** 当算法**检测到网络拥塞**时，它会**生成拥塞信号或标记**，通知网络中的发送方和接收方有关拥塞状态的信息。
3. **流量调整：** 算法会**根据拥塞信号调整流量**，以降低网络拥塞的程度。这可能包括减慢数据包的发送速率、实施流量控制策略等。
4. **拥塞窗口控制：** 一些算法使用拥塞窗口控制机制，该机制调整在任何给定时间内可以在网络上存在的未确认数据包的数量，以适应网络的容量。
5. **反馈机制：** 算法可能使用反馈机制，收集来自网络的信息，并将其用于调整流量控制参数，以更好地适应网络的动态变化。
6. **快速响应：** 尽管文本中提到端到端拥塞控制算法==可能需要至少1个往返时延（RTT）来对流量变化做出反应==，但这些算法通常会被设计为能够尽快地适应网络变化。

常见的端到端拥塞控制算法包括TCP（Transmission Control Protocol）的拥塞控制机制，如慢启动、拥塞避免和快速重传等。这些算法在互联网中的广泛应用有助于确保网络在高负载时仍能提供稳定的性能。在数据中心等环境中，也可能使用其他拥塞控制算法，以满足特定的网络需求。
###### 3. 端到端拥塞控制算法可能需要至少一个往返时延（RTT）来对流量变化做出反应，主要是由于以下原因：
1. **信息传播时延：** RTT代表了从发送端发送数据到接收端收到确认的时间。在拥塞控制中，发送方需要收到来自接收方的确认信息，以了解当前网络状态。由于RTT包括了数据传输、传播和处理的时间，信息传播的时延会影响拥塞控制的反应速度。
2. **往返：** ==拥塞控制是一个端到端的过程，需要在数据的发送端和接收端之间进行往返的通信==。**发送方发送数据，接收方收到数据后进行确认，并将确认信息返回给发送方**。这个==往返的过程需要至少一个RTT的时间。==
3. **动态网络环境：** 网络条件可能在时间上发生变化，例如带宽利用率、拥塞状态等。为了反映这些变化并采取相应的措施，拥塞控制算法需要一定的时间来收集足够的信息并做出适当的调整。RTT提供了一个基本的时间单位，用于测量网络环境的变化。
4. **流量调整的适度性：** 过于频繁地进行流量调整可能会导致网络不稳定和效率低下。通过使用RTT，拥塞控制算法可以更为适度地调整流量，避免在短时间内做出过于激进的反应。

尽管端到端拥塞控制算法在应对网络变化时可能需要等待至少一个RTT，但现代的拥塞控制算法通常会被设计为尽量减小这种延迟，以便更迅速地适应网络的动态变化。一些改进的算法，如快速重传、快速启动等，被引入以加速对拥塞的响应。 
###### 4. 对于短期拥塞事件，端到端拥塞控制可能无法进行有效的控制，原因主要包括以下几点：
1. **反应时间较慢：** 端到端拥塞控制的算法通常需要一定的时间来检测拥塞并做出调整。**短期拥塞事件的特点是发生和结束都很迅速**，而**端到端拥塞控制可能需要至少一个往返时延（RTT）来察觉拥塞并采取措施**。在这么短的时间内，端到端拥塞控制可能无法及时做出反应。
2. **拥塞窗口调整：** 端到端拥塞控制中一些算法，例如TCP的慢启动和拥塞避免，使用拥塞窗口来调整数据包的发送速率。在短期拥塞事件中，拥塞窗口可能没有足够的时间进行适度的调整，导致发送速率在拥塞事件发生和结束之间无法迅速适应。
3. **缺乏实时性：** 端到端拥塞控制通常设计用于处理长时间尺度上的拥塞，而不是短暂的拥塞事件。其设计理念可能不够关注短期变化，因此无法提供足够的实时性来有效地控制短期拥塞。
4. **网络环境的快速变化：** 在数据中心等环境中，网络可能因为应用之间的交互、任务调度等原因发生瞬时的拥塞。这种**快速的变化可能超过了端到端拥塞控制算法的响应速度**。

==为了更有效地应对短期拥塞事件，一些特定于数据中心网络的拥塞控制机制或者更为实时的反馈机制可能被引入==。这样的机制可能更加灵活和敏感，以适应数据中心网络中常见的短期拥塞情况。
#### 2. Within 1 RTT, it is the buffer management scheme that determines whether PFC messages can be avoided.
见上面✅
#### 3.  most congestion events will be caused by sub-RTT traffic bursts [19]
1. **拥塞事件：** 指的是网络中发生拥塞的情况，即网络中的流量过大，导致性能下降、数据包丢失或延迟增加等问题。
2. **亚往返时延（sub-RTT）：** 这里指的是==小于一个往返时延（RTT）的时间间隔==。RTT是从发送数据到接收确认所需的时间。因此，亚往返时延表示在一个 RTT 内的相对较短时间间隔。
3. **流量突发：** 指的是网络中突然涌入大量数据流量的情况，这可能是由于某些应用程序的临时性工作负载增加、任务调度变化等原因引起的。
    
综合起来，这句话表达了一个观点，即在网络中，==大多数拥塞事件是由于在相对短的时间内发生的流量突发，而这些流量突发的时间间隔小于一个往返时延==。这也强调了**对于短时间尺度上的拥塞事件**，特别是由于流量快速增加引起的事件，**端到端拥塞控制可能会面临一些挑战**。
#### 4. we find that the current headroom allocation scheme (which is called SIH in this paper) is quite inefficient in headroom utilization.
在您的描述中提到的"PFC交换机头部余量分配方案（SIH）"表明这是一种特定于使用PFC（Priority Flow Control）的交换机中的头部余量分配方案。让我对这个术语做一个基本的解释：
1. **PFC交换机：** PFC是一种用于以太网交换机的流控制机制，用于管理不同优先级的数据流，以防止缓冲区溢出和拥塞。在数据中心等环境中，PFC通常与高性能计算、存储等应用一起使用，以确保低延迟和高吞吐量。
2. **头部余量分配方案（SIH）：** "SIH" 可能是特定研究或文献中引入的术语，指代一种头部余量的分配策略。这种方案可能包括了为每个优先级或流量类别分配头部余量的方式，以应对不同优先级的流量需求。

具体的头部余量分配方案可能取决于具体的网络拓扑、PFC的配置和网络负载等因素。这些方案可能会被设计为更好地适应网络的特定需求，以提高性能和避免拥塞。
#### 5. This is because SIH reserves a static amount of worst- case headroom independently for every ingress queue in every port, which wastes a substantial amount of memory 
这段文字指出，“这是因为SIH（头部余量分配方案）在**每个端口的每个入口队列**==独立地==为最差情况==预留==了==静态数量的头部余量（headroom）==，这浪费了相当大的内存空间。”

具体解读如下：
1. **SIH头部余量分配方案：** SIH是头部余量分配方案的缩写，是一种用于管理头部余量的具体策略或算法。
2. **静态数量的头部余量：** SIH方案采用静态分配的方式，即为每个端口的每个入口队列预留了一个固定数量的头部余量。这种预留是基于最差情况的考虑，即在可能的最坏情况下保留的头部余量。
3. **独立分配：** SIH对于每个端口的每个入口队列都独立地进行头部余量的分配。这意味着每个队列都有自己独立的头部余量，而不受其他队列的影响。
4. **浪费大量内存：** 由于采用了静态分配，且为每个队列都独立分配了头部余量，这可能导致浪费相当大的内存空间。在实际网络使用中，可能并不是每个队列都在同一时间都需要预留最差情况下的头部余量，因此这种独立的、静态的分配方式可能导致内存资源的低效利用。
#### 6. However, the headroom reduction comes at a cost. Occupying the headroom reserved for each port will pause the entire traffic from the upstream link, which is harmful to performance as performance isolation is violated
###### 1. 详细解释：
这段文字表达了一种权衡：尽管减少头部余量可以带来一些好处，但却存在一定的代价。具体来说，==当占用为每个端口保留的头部余量时，会暂停来自上游链路的整个流量==，而这对性能造成了不利的影响，因为性能隔离受到了侵犯。

详细解释如下：
1. **头部余量减少的代价：** 文中提到 "the headroom reduction comes at a cost" 意味着尽管减少头部余量可能有一些好处，但与此同时也伴随着某种代价。
2. **占用头部余量会暂停整个流量：** ==当某个端口占用了为其保留的头部余量时，文中提到会“暂停整个流量”，这意味着来自上游链路的流量将会停止传输==。**头部余量通常用于处理拥塞或临时的流量突发，但当它被占用时，系统可能会采取某种方式来停止流量，以防止可能的拥塞**。
3. **对性能的不利影响：** 文中提到这种暂停整个流量的操作“对性能造成了不利的影响”，这是因为在这个过程中，性能隔离被违反了。==性能隔离通常是指在一个端口或一个流量类别的影响下，不应该影响其他端口或流量类别。但当整个流量被暂停时，这种隔离被打破，可能导致性能下降、延迟增加等问题。==
4. **权衡：** 这段文字传达的是一种权衡，即在追求减少头部余量的同时，需要考虑可能的代价，即暂停整个流量可能对性能造成的负面影响。这样的权衡是在设计网络或系统时需要综合考虑的重要因素。
总的来说，这段文字强调了头部余量管理中的一个问题，即在一些情况下减少头部余量可能导致性能隔离的破坏，需要综合考虑权衡取舍。
###### 2. 为什么当每个端口保留的头部余量被占用时，会暂停来自上游链路的整个流量？
###### （翻译问题，这个问题很显然，因为headroom就是为“grey period”兜底的）
当占用为每个端口保留的头部余量时，会暂停来自上游链路的整个流量，这通常涉及到使用了一种流量控制机制，可能是 PFC（Priority Flow Control）或类似的机制。这种暂停的原因与网络中的流量控制和拥塞控制有关。以下是一些可能的原因：
1. **PFC机制：** 如果网络中使用了==PFC机制==，该机制==允许端口通过发送PFC帧来暂停特定优先级的流量==。PFC通常与Ethernet中的802.1Qbb标准一起使用，它允许通过发送PFC帧来暂停或恢复具有特定优先级的数据流。这种机制的==目的是防止缓冲区溢出和拥塞==。
2. **头部余量用于PFC：** *头部余量通常用于处理拥塞和流量管理*。==当为每个端口保留的头部余量被占用时，可能是因为PFC机制触发了==，要求暂停来自上游链路的整个流量。这可能是为了防止缓冲区溢出，特别是在拥塞或高优先级流量涌入时。
3. **性能隔离和拥塞控制：** 这种暂停的操作可能是为了维持性能隔离和进行拥塞控制。当某个端口的流量超出了预留的头部余量，系统可能会采取措施以确保这个端口的流量不会对其他端口或链路产生不利影响。

总的来说，这种暂停的行为是出于网络性能和拥塞控制的考虑，以确保在某个端口或链路上发生拥塞时，采取的措施不会影响整个网络的稳定性和性能。这种机制有助于防止拥塞扩散，并确保网络的可靠性。