# Evaluation

In this section, we evaluate DSH’s performance with extensive packet-level simulations on the ns-3 platform [44]. We summarize the results below.
在这一部分中，我们通过在ns-3平台上进行大量的分组级模拟来评估DSH的性能[44]。我们将结果总结如下。

- DSH can absorb 4× more bursty traffic without triggering PFC messages. 
- DSH can effectively mitigate the performance impair-ments (i.e., collateral damage and deadlock) brought by PFC messages. 
- DSH can reduce the ==FCT== by up to 57.7% for ==short fan-in flows== and up to 31.1% for ==one-to-one background flows== in large-scale DCN topology.

- DSH能够吸收4倍以上的突发流量，而不触发PFC消息。
- DSH能够有效减轻由PFC消息引起的性能损害（即，附带损害和死锁）。
- DSH在大规模数据中心网络拓扑中，对 *短fan-in流* 和 *一对一后台流* 的**FCT**能够分别减少最多57.7%和31.1%。
###### ps：FCT
>在计算机网络中，FCT（Flow Completion Time）是指**完成一个数据流传输所需的时间**。它是衡量网络性能的重要指标，涉及到 *从发送端发送数据* 到 *接收端完成接收* 的整个时间过程。FCT包括了多个阶段，如数据传输延迟、传输队列等待时间以及其他可能的延迟因素。
###### PS：“short fan-in flows" 和 "one-to-one background flows" 
>在上下文中，"short fan-in flows" 和 "one-to-one background flows" 是关于网络流量模型的描述。
>1. **Short Fan-in Flows:**
>- "Short fan-in flows" 意味着这是一种流量模型，其中涉及到==短暂且汇聚（fan-in）的数据流==。
>- "Short" 表示这些数据流的持续时间相对较短。
>- "Fan-in" 表示这些流量源自不同的入口（ingress）端口，但最终汇聚到同一个出口（egress）端口。
>2. **One-to-One Background Flows:**
>- "One-to-one background flows" 意味着这是一种流量模型，其中每个后台流都与另一个特定的流相对应。
>- "One-to-one" 表示每个后台流与一个特定的目标流相对应，可能表示每个后台流有一个独立的通信关系。
>- =="Background flows" 通常指的是那些不太紧急或不太重要的流量==，可能是系统中的一些常规通信。
>
在你提到的上下文中，作者可能使用这些术语来描述他们在评估DSH性能时所使用的流量模型，以便更清晰地理解系统在处理不同类型流量时的行为和性能表现。

FCT的计算通常包括以下主要组成部分：
1. **发送延迟（Send Delay）：** 从数据流开始发送到第一个比特离开发送端所经历的时间。
2. **传输延迟（Transmission Delay）：** 数据在传输介质中传播所需的时间，通常与带宽和距离有关。
3. **排队延迟（Queueing Delay）：** 数据在网络中等待传输的时间，与网络拥塞程度和队列长度有关。
4. **处理延迟（Processing Delay）：** 数据在路由器或交换机中进行处理所需的时间，包括路由计算、查找转发表等操作。
5. **接收延迟（Receive Delay）：** 接收端从接收到数据的第一个比特到整个数据流被接收完成所需的时间。

综合考虑上述延迟，FCT的计算公式可以表示为：
$FCT = \text{发送延迟} + \text{传输延迟} + \text{排队延迟} + \text{处理延迟} + \text{接收延迟}$
优化FCT是网络设计和管理中的重要目标，特别是在数据中心网络等对性能要求较高的场景中。减少FCT有助于提高网络的响应性、降低传输延迟，从而改善用户体验和应用性能。
### A. Microbenchmarks（微基准测试）
###### ps：微基准测试
微基准测试指的是针对*系统或组件* **特定且孤立**方面进行评估的**小规模性能测试**。这些测试旨在测量单个功能、模块或操作的性能，以识别潜在瓶颈、评估效率，并了解系统在特定条件下的行为。微基准测试对于精细粒度的性能分析和优化非常有价值。

微基准测试的关键特点包括：
1. **隔离性：** 微基准测试隔离了特定的功能或组件，允许有针对性地评估，而不受其他系统元素的干扰。
2. **精确性：** 这些基准测试旨在精确测量特定功能或操作的性能，提供对其效率和执行特性的详细了解。
3. **可重现性：** 微基准测试通常被设计为易于重现，确保重复测试能够产生一致的结果。
4. **关注度量标准：** 它们通常关注定量指标，如执行时间、吞吐量、延迟或资源利用率，以量化目标方面的性能。
5. **简单性：** 微基准测试通常相对简单，因为它们旨在孤立评估特定功能。这种简单性有助于分析特定组件，而无需整个系统的复杂性。

微基准测试广泛用于软件开发、系统优化和性能调整。它们在识别性能瓶颈、验证优化工作以及指导软件和硬件设计方面发挥着关键作用。此外，这些基准测试对比较不同算法实现或版本，以及评估变更对系统特定组件的影响都非常有价值。

In this part, we evaluate DSH’s basic performance with care-fully constructed microbenchmarks. We emulate the Broadcom Tomahawk switching chip, which has 32 100Gbps ports and 16MB shared memory. Each port has 8 queues. One queue is reserved for ACK packets and PAUSE/RESUME messages, and has the highest priority. Other seven queues are scheduled by the DWRR algorithm with a quantum of 1600B. The link delay is 2μs and thus η = 56840B. The total headroom size for SIH is 56840B × 32 × 7=12MB. The private buffer size is 672KB (3KB for each DWRR-scheduled queue). For DT, α is set to 1/16 according to [3]. The Xqon/Xpon threshold is the same as the Xqoff /Xpoff threshold.

在这一部分，我们通过精心构建的微基准测试评估了DSH的基本性能。
我们模拟了Broadcom Tomahawk交换芯片，该芯片具有32个100Gbps端口和16MB共享内存。
每个端口有8个队列。**其中一个队列用于ACK数据包和PAUSE/RESUME消息，具有最高优先级。** 
其他七个队列由==DWRR算法==调度，每个队列的权重为1600B。
链路延迟为2μs，因此η = 56840B。
SIH的总储备空间大小为56840B × 32 × 7 = 12MB。
私有缓冲区大小为672KB（每个DWRR调度队列为3KB）。
对于DT，根据[3]，α设置为1/16。Xqon/Xpon阈值与Xqoff/Xpoff阈值相同。

###### ps：DWRR_Algorithm
>DWRR（Deficit Weighted Round Robin）算法是一种流量调度算法，常用于网络中的队列管理。它的设计目标是通过动态调整权重来实现公平性和灵活性，确保资源分配更符合流量的需求。以下是DWRR算法的基本原理：
>1. **权重分配：** 每个队列都被分配一个权重值，表示其在调度时获得服务的相对优先级。较高权重的队列在每轮调度中获得更多的服务机会。
>2. **缺额计数：** 每个队列都有一个缺额计数器，初始化为零。在每轮调度中，队列未使用的权重将被累积到缺额计数器中
>3. **调度过程：** 在每个调度周期中，DWRR按照队列的权重依次为每个队列分配服务机会。队列被允许发送的数据量由其缺额计数决定。每个队列在获得服务机会后，其缺额计数将被减去其权重值。
>4. **动态调整：** 如果某个队列未用完其服务机会，剩余的权重将被保留到下一轮调度。这使得DWRR能够动态适应流量的变化，更好地处理不同优先级和带宽需求的流。
>
在文中提到的情境中，DWRR算法被用于调度Broadcom Tomahawk交换芯片上的七个队列，每个队列的权重为1600B。这样的设置允许对不同类型的数据包进行公平调度，确保高优先级队列能够更及时地获得服务。

> https://blog.csdn.net/lsl_zhulin/article/details/115643382

**PFC avoidance：** First, we evaluate the ability of DSH to avoid PFC messages. As shown in Fig. 11a, we start two background flows at the beginning, which are from ingress port 0 and ingress port 1, respectively, and head for egress port 31. At t =0.1s, we start 16 fan-in bursty flows, which are from ingress port 2-17 to egress port 30. Fig. 11b shows the total pause duration of all fan-in flows. DSH can avoid PAUSE messages when the burst size is no larger than 40% of buffer size, which is over 4× better than SIH.学术化翻译成中文

PFC（Priority Flow Control）的规避。首先，我们评估DSH规避PFC消息的能力。如图11a所示，我们在开始时启动了两个后台流[background flows]，分别来自入口端口0和入口端口1，目的地是出口端口31。在t=0.1s时，我们启动了16个短时突发流[fan-in flows]，它们从入口端口2-17到达出口端口30。图11b显示了所有短时突发流的总暂停持续时间。当突发大小不超过缓冲区大小的40%时，DSH可以避免PAUSE消息，这比SIH好了4倍以上。

![[Pasted image 20231126154314.png]]

**Deadlock avoidance：** One impairment brought by PFC mes-sages is deadlock [3], [6]–[8], [11], which is a serious problem as it can make a large part of network unusable. In this part, we evaluate the ability of DSH to avoid deadlock. We consider a topology shown in Fig. 12a, which is a leaf-spine topology with two link failures marked with dashed lines (i.e., S0-L3 and S1-L0). In the topology, there are two spine switches (S0 and S1) and four leaf switches (L0 − L3). Each leaf switch is connected to 16 hosts with 100Gbps downlinks, and connected to two spine switches with 400Gbps uplinks. The link delay is 2μs. We generate fan-in flows, which are from L0 to L3, from L3 to L0, from L1 to L2, and from L2 to L1, respectively. As a result, there is a cyclic buffer dependency marked as red lines: S0 → L1 → S1 → L2 → S0. The fan-in ratio (i.e., the number of flows) ranges from 1 to 15. The flow size is randomly chosen based on the Hadoop workload [28], and flow arrivals follow a Poisson process. The network load is 0.5 at the downlinks of each leaf switch. Each scheme is tested 100 times and each simulation lasts for 100ms. Fig. 12b shows the CDF of deadlock onset time. With SIH, deadlock occurs for all simulations either with DCQCN [1] or PowerTCP [37]. In comparison, DSH can avoid 96% deadlocks with DCQCN and all deadlocks with PowerTCP. This is because DSH can leave more “footroom” to absorb bursty traffic, avoiding PFC messages.

死锁规避。PFC（Priority Flow Control）消息引入的一个问题是死锁[3]，[6]–[8]，[11]，这是一个严重的问题，因为它可能导致网络的大部分无法使用。在这一部分，我们评估DSH规避死锁的能力。

我们考虑了图12a中显示的拓扑结构，这是一个具有**两个虚线标记的链路故障的叶脊拓扑结构（即，S0-L3和S1-L0）**。
在拓扑结构中，有两个脊交换机（S0和S1）和四个叶交换机（L0 − L3）。
每个叶交换机连接到16个具有100Gbps下行链路的主机，并通过400Gbps上行链路连接到两个脊交换机。
链路延迟为2μs。我们生成汇聚流，分别从L0到L3，从L3到L0，从L1到L2，以及从L2到L1。因此，存在一个被标记为红线的循环缓冲区依赖关系：S0 → L1 → S1 → L2 → S0。 
汇聚比率（即:流的数量）从1到15不等。流量大小==基于Hadoop工作负载[28]随机选择==，==流到达遵循泊松过程==。每个叶交换机下行链路的网络负载为0.5。每个方案测试100次，每次模拟持续100毫秒。

图12b显示了死锁发生时间的累积分布函数。对于SIH，无论使用DCQCN[1]还是PowerTCP[37]，都会在所有模拟中发生死锁。相比之下，DSH在使用DCQCN时可以避免96%的死锁，在使用PowerTCP时可以避免所有死锁。这是因为DSH可以留出更多的“余地”来吸收突发流量，从而避免PFC消息。

![[Pasted image 20231126160536.png]]
###### PS：flow遵从poisson分布
>"流到达遵循泊松过程"指的是流量到达的模式，其中流的发生符合泊松过程。泊松过程是一种随机过程，描述了一系列独立事件在时间上的发生，且其发生的间隔时间遵循指数分布。
>
在网络领域中，泊松过程通常用于模拟流量的到达模式。具体而言，如果流到达遵循泊松过程，这意味着流量的到达时间是随机的且事件之间的间隔时间是指数分布的。以下是关于泊松过程的一些关键概念：
【1】**独立性：** 在泊松过程中，事件的发生是相互独立的，一个事件的发生不会影响其他事件的发生。
【2】**恒定速率：** 泊松过程假设事件的平均发生速率是恒定的，即在任何固定时间段内发生事件的概率是相同的。
【3】**指数分布：** 事件之间的时间间隔（到达时间）在泊松过程中遵循指数分布。指数分布的概率密度函数（PDF）表现为急剧下降，适合描述短时间内多次事件发生的情况。
>>在提到流到达遵循泊松过程时，作者可能在描述流量到达的模型，其中流的到达时间是随机的，且到达之间的间隔时间符合指数分布。这种模拟可以用于评估网络在实际运行环境中面对随机性流量时的性能表现。
###### PS：为什么网络领域中，泊松过程通常用于模拟流量的到达模式
>在==网络领域中，泊松过程通常用于模拟流量的到达模式==，原因如下：
>1. **独立性和恒定速率：** 泊松过程的两个关键特性是*事件的独立性* 和*恒定速率*。这两个特性使得泊松过程成为模拟随机事件到达的理想选择。在网络中，例如，用户请求、数据包到达、任务提交等事件可能具有独立性，并且其平均到达速率可能在一定时间范围内是相对稳定的。这样的模型对于描述这些随机事件的到达模式是合适的。
>2. **数学简单性：** 泊松过程具有*数学上的简单性*，其概率分布函数和累积分布函数都有封闭形式的表达式。这使得对于这种类型的过程进行分析和计算变得相对容易。
>3. **指数分布特性：** *泊松过程中事件到达的时间间隔符合指数分布*。指数分布的特性使得*模型更容易处理*，而且对于描述随机事件的到达时间具有合理的数学基础。
>4. **适用性广泛：** 泊松过程不仅在网络领域中被广泛使用，还在其他领域，如排队理论、通信系统、生物学等方面得到了广泛应用。这种广泛的适用性使得人们能够更容易地将各种系统的性能建模与分析与泊松过程相结合。
>
总体而言，泊松过程为模拟和研究网络中的随机事件提供了一种简单而有效的数学工具。虽然实际网络中的流量往往更为复杂，但泊松过程提供了一个初始的近似，对于理解网络性能、评估系统容量和进行性能分析仍然是有用的。

**Collateral damage mitigation**：Another impairment brought by PFC messages is that it can lead to performance degradation of innocent flows, which is known as collateral damage [3]. In this part, we show that DSH can mitigate this problem by avoiding PFC PAUSEs. We consider a typical scenario shown in Fig. 13a, which is a common unit in datacenters and widely adopted by other work [9], [45], [46]. All links are 100Gbps and the propagation delay is 2μs. Two long-lived flows, F0 and F1, are sending traffic from H0 and H1 to R0 and R1, respectively. After their throughputs reach 50Gbps, H2-H25 generate 24 concurrent fan-in flows to R1. Each flow has a size of 64KB, which is smaller than a BDP and thus the fan-in traffic is uncontrolled by congestion control algorithms. At this time, the congestion point is S1 and both F1 and fan-in flows contribute to the congestion. On the other hand, F0 is an innocent flow that does not contribute to the congestion. Ideally, the throughput of F0 should not be reduced. Fig. 13 shows the throughput of F0. With SIH, the through-put of F0 is significantly reduced. This is because the pause threshold Xpoff is very low and thus PFC PAUSEs can be easily triggered. As a result, switch S0 is paused and the packet transmission of F0 is suspended. In comparison, DSH can effectively avoid performance degradation for F0. This is because it can efficiently utilize the headroom and leave enough “footroom” for PFC avoidance. Besides, we observe that the state-of-the-art congestion control algorithms (Fig. 13c and Fig. 13d) are not able to avoid the collateral damage. This is because end-to-end congestion control requires at least 1 RTT to react to traffic changes. Within 1 RTT, it is the buffer management scheme that determines whether PFC messages can be avoided.

缓解附带损害：PFC消息引起的另一个问题是它可能导致无辜流的性能下降，这被称为"collateral damage"（附带损害）[3]。
在这一部分，我们展示了DSH通过避免PFC PAUSE可以缓解这个问题。
我们考虑了图13a中显示的典型场景，这是数据中心中常见的单元，也被其他研究广泛采用[9]，[45]，[46]。
所有链路速度为100Gbps，传播延迟为2μs。
两个长寿命流，F0和F1，分别从H0和H1发送流量到R0和R1。
在它们的吞吐量达到50Gbps后，H2-H25生成了24个并发的汇聚流到达R1。
每个流的大小为64KB，==小于**BDP**，因此汇聚流量不受拥塞控制算法的控制==。
此时，拥塞点是S1，F1和汇聚流都导致了拥塞。另一方面，F0是一个无辜流，不会导致拥塞。
理想情况下，F0的吞吐量不应减少。
图13显示了F0的吞吐量。使用SIH时，F0的吞吐量显著减少。这是因为暂停阈值Xpoff非常低，因此可以很容易地触发PFC PAUSE。结果，交换机S0被暂停，F0的数据包传输被中止。
相比之下，DSH可以有效避免F0性能下降。这是因为它可以高效利用头部空间，并为PFC规避留出足够的"余地"。此外，我们观察到现代拥塞控制算法（图13c和图13d）不能避免附带损害。这是因为端到端的拥塞控制需要至少1个往返时间（RTT）来对流量变化做出反应。在1个RTT内，确定是否可以避免PFC消息的是缓冲区管理方案。
###### PS：Collateral damage mitigation
>- "Collateral damage mitigation" 意指**减轻或缓解在某个过程、决策或操作中产生的附带损害**。在特定的上下文中，"collateral damage"（附带损害）是指在实施某项决策或操作时可能对其他方面产生的不良影响或额外的损害。因此，"collateral damage mitigation" 强调采取措施来最小化或避免这些附带的不良影响。
>- 在提到网络中的 "collateral damage mitigation" 时，可能指的是采取一些策略或设计来减轻在网络操作中产生的附带损害，==特别是在涉及到 PFC（Priority Flow Control）消息引起的问题时。在网络通信中，一些优化或调整可能会影响到整体网络性能，尤其是可能对不相关的流量或任务造成不利影响==。在这种情况下，==采取措施以减轻或避免这些额外的不良影响就是 "collateral damage mitigation" 的目标==。
###### PS：BDP
>"BDP" 是网络中的一个术语，表示**带宽延迟积**（Bandwidth-Delay Product）。带宽延迟积是指在网络通信中，==通过乘以带宽（数据传输速率）和往返时间（Round-Trip Time, RTT），得到一个衡量 *网络容量* 的值==。数学上，BDP可以用以下公式表示：
>>$BDP = \text{带宽} \times \text{往返时间 (RTT)}$
>
在上述上下文中，提到每个流的大小为64KB，小于 BDP，意味着每个流的数据量相对较小，不足以充分利用网络的带宽容量。这种情况下，流的大小不会导致网络出现拥塞，因为其数据量未达到网络在给定时间内可以容纳的最大数据量（由BDP决定）。
当流的大小小于 BDP 时，就说明网络在单位时间内能够容纳更多的数据，因此拥塞控制算法可能不会被触发。拥塞控制算法通常根据网络的负载情况来调整数据的发送速率，以避免网络拥塞。在这个特定的场景中，由于流的大小相对较小，可能认为它们不太可能对网络产生明显的拥塞效应，因此拥塞控制算法可能不会介入对这些流进行调整。
###### PS：拥塞控制算法被触发的条件
>拥塞控制算法被触发通常需要满足以下一些条件：
>1. **网络负载高：** 当网络中的数据流量较大，接近或达到网络容量的上限时，可能会发生拥塞。拥塞控制算法会监测网络的负载情况，当网络负载达到一定的阈值时，就可能触发拥塞控制机制。
>2. **队列溢出：** 在网络设备（如路由器或交换机）中，数据包通常会存储在缓冲队列中等待传输。如果这些队列变得过于拥挤，导致队列溢出，就可能触发拥塞控制算法。队列溢出是拥塞的一个指示，表明网络设备无法及时处理所有传入的数据。
>3. **丢包率升高：** 拥塞通常与数据包的丢失相关。当网络设备无法处理所有到达的数据包时，可能会出现丢包。拥塞控制算法可以根据丢包率的升高来判断网络是否处于拥塞状态，并采取相应的措施减缓数据传输速率。
>4. **延迟增加：** 拥塞还可能导致网络的往返时间（Round-Trip Time, RTT）增加。拥塞控制算法会监测延迟的变化，当延迟增加到一定程度时，可能会启动拥塞控制机制。
>5. **ECN（显式拥塞通知）标记：** 在一些网络中，路由器或交换机可以使用 ECN 标记来指示网络的拥塞状态。当数据包经过某个拥塞的网络节点时，该节点可以设置 ECN 标记。拥塞控制算法可以根据 ECN 标记来判断网络状态，并相应地进行调整。
>
这些条件的组合可能因网络设计和拥塞控制算法的具体实现而有所不同。拥塞控制算法的目标是在网络负载高的情况下维持稳定的性能，并避免因过度拥塞而导致的性能下降。

![[Pasted image 20231126163009.png]]
### B. Benchmark Traffic（基准流量）
>在网络领域中，"Benchmark Traffic"（基准流量）通常指用于**性能评估和测试**的==标准化流量模型==。这种流量模型设计得足够简单和具有代表性，以便评估网络设备、协议或系统的性能。基准流量可用于比较不同系统之间的性能，进行基准测试和验证系统在不同负载条件下的表现。
>
基准流量的特点可能包括：
>1. **已知流量模式：** 基准流量通常具有**已知的流量模式**，例如特定大小的数据包、特定的到达率或流的数量。这有助于在测试中精确地控制和测量流量。
>2. **可重现性：** 基准流量的生成应**具有可重现性**，以确保在不同的测试环境中能够产生相似的流量，从而实现可靠的比较。
>3. **典型用例：** 基准流量通常设计为模拟网络中的**典型使用**情况，以便评估系统在实际工作负载下的表现。
>4. **容易分析：** 基准流量通常设计为**易于分析**，以便评估网络设备或系统的性能特征，例如吞吐量、延迟和丢包率。
>5. **代表性：** 基准流量应当**具有代表性**，反映出网络中常见的流量特征，以确保测试结果对于实际应用场景具有一定的可靠性。
>>在研究和测试中，基准流量被广泛用于评估各种网络技术和解决方案的性能。这种流量可以是事先定义的，也可以根据特定的测试需求进行定制。

In this part, we evaluate DSH in a large-scale DCN topology.
在这一部分，我们对DSH在一个大规模的数据中心网络（DCN）拓扑结构中的性能进行评估。
##### Topology. 
We build a leaf-spine topology with 16 leaf switches, 16 spine switches, and 256 servers. Each leaf switch is connected to 16 servers with 100Gbps downlinks and 16 spine switches with 100Gbps uplinks, forming a full-bisection network. The link delay is 2μs and thus the base RTT is 16μs across the spine. We employ ECMP for load balancing.

拓扑结构。我们构建了一个包含16个叶交换机、16个脊交换机和256个服务器的叶脊拓扑结构。
每个叶交换机通过100Gbps下行链路连接到16台服务器，并通过100Gbps上行链路连接到16个脊交换机，形成一个全分割网络。
链路延迟为2μs，因此沿脊的基本往返时间（RTT）为16μs。我们采用ECMP进行负载均衡。
##### Switch. 
We emulate the Broadcom Tomahawk switching chip. The settings are the same as those in previous simulations.

交换机。我们模拟了Broadcom Tomahawk交换芯片，其设置与之前的仿真相同。
##### Transport. 
We consider two end-to-end congestion control algorithms: DCQCN [1] and PowerTCP [37]. We use the default parameter settings in their open-source simulations.

传输。我们考虑了两种端到端拥塞控制算法：DCQCN [1] 和PowerTCP [37]。我们使用它们在开源仿真中的默认参数设置。
##### Workload. 
We generate two kinds of traffic: background traffic and bursty fan-in traffic. The background traffic follows one-to-one pattern. The sender and receiver are randomly chosen. The flow sizes are generated according to a web search workload [27]. The fan-in traffic follows many-to-one pattern, where 16 senders simultaneously transmit 64KB data to the same receiver. The senders are randomly chosen and are in different racks from the receiver. Fan-in flows are classified into the same traffic class. Background flows are randomly classified into other traffic classes. Flow arrivals follow a Poisson process. The total network load is 0.9.

工作负载。我们生成两种类型的流量：后台流量（background flows）和 突发的汇聚流量（fan-in flows）。
后台流量遵循一对一的模式。发送方和接收方是随机选择的。流大小根据Web搜索工作负载[27]生成。
汇聚流量遵循多对一的模式，其中16个发送方同时向同一接收方传输64KB的数据。发送方是随机选择的，并且位于接收方不同的机架中。
汇聚流被分类为相同的流量类别。后台流被随机分类为其他流量类别。流到达遵循泊松过程。总网络负载为0.9。

![[Pasted image 20231126165516.png]]
##### Results.
Fig. 14 shows the flow completion time (FCT) under different loads of background traffic. For clear comparison, we normalize each FCT to the value achieved by SIH. The results show that both fan-in flows and background flows can benefit from DSH. With DCQCN, DSH can reduce the average FCTs of background traffic and fan-in traffic by up to 10.1% and 43.3%, respectively. With PowerTCP, DSH can reduce the average FCTs of background traffic and fan-in traffic by up to 31.1% and 57.7%, respectively. This is because DSH can provide more footroom to absorb transient bursty traffic and avoid PFC messages. Furthermore, we evaluate DSH across different DCN ap-plications and network architectures. Besides the web search workload, we also consider other three realistic workloads: a data mining workload [47], a cache workload [28], and a Hadoop workload [28]. Besides leaf-spine topology, we also consider fat-tree topology (k=16) [48]. Other settings keep unchanged. Fig. 15 shows the FCT of background traffic across different workloads and topologies with DCQCN. The results confirm that DSH can improve FCT with different DCN workloads and network topologies.

结果。图14显示了在不同背景流量负载下的**流完成时间（FCT）**。
为了进行清晰比较，我们将每个FCT标准化为SIH实现的值。
结果显示，DSH对汇聚流和后台流均能产生益处：
- 使用DCQCN，DSH可以将后台流量（bgd flow）和汇聚流量（fan-in flow）的平均FCT分别降低10.1%（b）和43.3%（a）。
- 使用PowerTCP，DSH可以将后台流量和汇聚流量的平均FCT分别降低31.1%（d）和57.7%（c）。

这是因为DSH可以提供更多余地来吸收瞬态的突发流量并避免PFC消息。

![[Pasted image 20231126170101.png]]

此外，我们在不同的DCN应用和网络架构中评估了DSH。除了Web搜索工作负载外，我们还考虑了其他三种真实的工作负载：数据挖掘工作负载[47]、缓存工作负载[28]和Hadoop工作负载[28]。除了叶脊拓扑结构外，我们还考虑了脂肪树拓扑结构（k=16）[48]。其他设置保持不变。

图15显示了在不同工作负载和拓扑结构下使用DCQCN的后台流量的FCT。结果证实了DSH在不同的DCN工作负载和网络拓扑结构下都能改善FCT。
### 附注：
#### 数据挖掘工作负载

#### 缓存工作负载

#### Hadoop工作负载

#### 叶脊拓扑结构
DCN（数据中心网络）的叶脊拓扑结构是一种广泛用于构建数据中心网络的设计。这种拓扑结构的目标是提供高带宽、低延迟和高度可扩展性，以满足大规模数据中心的需求。

在叶脊拓扑结构中，网络由两种类型的交换机组成：*叶交换机（Leaf Switches）* 和 *脊交换机（Spine Switches）*。以下是叶脊拓扑结构的基本特征：
1. **叶交换机：** 叶交换机连接到数据中心内部的服务器。**每个叶交换机通常连接到多个服务器，形成了一个聚合点**。这些连接通常是高带宽的，例如100Gbps或更高。
2. **脊交换机：** **脊交换机则连接到所有的叶交换机**，形成一个脊与叶的网状结构。这样的设计使得任意两个服务器之间都有多条路径可选，提高了网络的冗余性和可用性。脊交换机之间的连接也通常是高带宽的。
3. **全双工通信：** 在叶脊拓扑结构中，通常采用**全双工通信**，允许数据同时在两个方向上传输，提高了网络的吞吐量。
4. **低延迟：** 叶脊拓扑结构的设计旨在保持**低延迟**。由于存在**多路径**，数据可以选择最短路径传输，减少了传输延迟。
5. **可扩展性：** 叶脊拓扑结构是**可扩展**的，可以轻松地添加更多的叶交换机和脊交换机来增加网络的规模，适应不断增长的数据中心需求。

总体而言，叶脊拓扑结构是一种有效的数据中心网络设计，具有高性能、可扩展性和冗余性，使其成为大规模数据中心的理想选择。
#### 脂肪树（fat-tree）拓扑结构（k=16）
##### 简介：
脂肪树是一种用于构建数据中心网络的拓扑结构，旨在提供高性能、低延迟和高度冗余的网络连接。它被广泛应用于大规模数据中心，支持大量服务器之间的通信。
##### 主要特征：
1. **多层结构：** 脂肪树拓扑结构通常包含多个层次，其中包括核心（Core）、汇聚（Aggregation）、边界（Edge）和端点（Leaf）层。每个层次都有其特定的功能和连接。
2. **全双工通信：** 脂肪树网络中通常采用全双工通信，允许同时进行双向数据传输，提高了网络的吞吐量。
3. **冗余路径：** 脂肪树的设计使得任意两个节点之间存在多条冗余路径，提高了网络的冗余性和可用性。即使某些路径发生故障，仍然有备用路径可用。
4. **低延迟：** 由于存在多条路径，数据可以选择最短路径传输，从而减少了传输延迟。
5. **可扩展性：** 脂肪树是可扩展的，可以通过增加更多的交换机和连接来扩展网络规模，适应不断增长的数据中心需求。
6. **容错性：** 脂肪树的冗余性和多路径设计提供了一定程度的容错性，使网络更具稳定性。

总体而言，脂肪树拓扑结构是一种优秀的数据中心网络设计，提供了高性能、冗余性和可扩展性，适用于大规模的计算和存储需求
##### 具体过程：
Fat-Tree是以交换机为中心的拓扑。支持在横向拓展的同时拓展路径数目；且所有交换机均为相同端口数量的普通设备，降低了网络建设成本。

Fat-Tree结构共分为三层：核心层、汇聚层、接入层。一个k元的Fat-Tree可以归纳为5个特征：

每台交换机都有k个端口；
1. 核心层为顶层，一共有(k/2)^2个交换机；
2. 一共有k个pod，每个pod有k台交换机组成。其中汇聚层和接入层各占k/2台交换机；
3. 接入层每个交换机可以容纳k/2台服务器，因此，k元Fat-Tree一共有k个pod，每个pod容纳k*k/4个服务器，所有pod共能容纳k*k*k/4台服务器；
4. 任意两个pod之间存在k条路径。

常见的有2元、4元、6元等结构：
![[Pasted image 20231126173115.png]]
![[Pasted image 20231126173125.png]]
