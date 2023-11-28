# DSH_Design

To address the inefficiency problem of SIH, we propose Dynamic and Shared Headroom (DSH) allocation, which aims to efficiently allocate headroom while ensuring no congestion loss. In this section, we first explain the key ideas behind DSH (§IV-A). Then we present the details of our design (§IV-B), and theoretically analyze DSH’s ability of burst absorption (§IV-C). Finally, we show that DSH is easy to be implemented on switching chips (§IV-D).

>为了解决SIH（Static and Individual Headroom）的低效问题，我们提出了==动态和共享的头部空间（Dynamic and Shared Headroom，DSH）分配方案==，旨在*有效分配头部空间* 的同时*确保没有拥塞丢失* 。
在本节中，我们首先解释DSH背后的**关键思想**（§IV-A）。然后，我们介绍我们**设计的详细内容**（§IV-B），并理论上分析**DSH对于吸收突发流量的能力**（§IV-C）。最后，我们**展示DSH在交换芯片上易于实现**（§IV-D）。

### A. Key Ideas
DSH utilizes two ideas to achieve efficient headroom allo- cation while ensuring no congestion loss. 
DSH采用两个思想来实现高效的头部空间分配，同时确保没有拥塞丢失。

(1) DSH proactively reserves a small amount of buffer as insurance headroom to avoid packet loss. Different ingress queues in the same port share the uplink capacity. Thus, to avoid buffer overflow under any circumstances, there is no need to allocate η headroom for each ingress queue. Rather, we only need to reserve η buffer for each port. When any ingress queue in a port starts to occupy the insurance buffer, the port pauses the entire upstream port. In this way, the amount of reserved headroom is significantly reduced. Of course, this can violate performance isolation among different traffic classes. Thus, we also have the following mechanism to make the port- wise pause rarely triggered. 

>（1）DSH主动==保留一小部分缓冲作为保险头部空间==，以避免数据包丢失。同一端口中的不同入口队列共享上行容量。因此，为了在任何情况下都避免缓冲溢出，==没有必要为每个入口队列分配 η 头部空间==。相反，我们==只需要为每个端口保留 η 缓冲==。==当端口中的任何入口队列开始占用保险缓冲时，端口将暂停整个上行端口==。通过这种方式，保留头部空间的量得到了显著减少。当然，这可能会违反不同流量类之间的性能隔离。因此，我们还有以下机制，使得端口暂停很少触发。

(2) DSH dynamically allocates the headroom to congested queues and makes the allocated headroom shared among different ingress queues. An ingress queue needs to occupy headroom only when it is congested. Thus, rather than wasting headroom for uncongested queues, DSH tries to allocate headroom to a queue only when it becomes congested. In this way, when few queues are congested, most buffer can serve as “footroom” to absorb bursty traffic without triggering PFC pauses. Furthermore, as the allocated headroom buffer is not necessarily used up, DSH enables the headroom buffer to be shared among ingress queues. In this way, DSH can take advantage of statistical multiplexing to improve the buffer efficiency.

>（2）DSH动态==分配头部空间给拥塞的队列==，并==使分配的头部空间在不同的入口队列之间共享==。只有在入口队列拥塞时，它才需要占用头部空间。因此，与其为非拥塞队列浪费头部空间，**DSH试图只在队列拥塞时为其分配头部空间**。通过这种方式，当很少的队列拥塞时，大多数缓冲可以用作（**footroom**）“**余量空间**”，用于吸收突发流量而不触发PFC暂停。此外，由于分配的头部空间缓冲未必会用完，DSH使头部空间缓冲能够在入口队列之间共享。通过这种方式，DSH可以利用统计复用提高缓冲效率。

### B. DSH Mechanisms

![[Pasted image 20231125192801.png]]
#### Buffer organization：
Fig. 7 shows the buffer organization with DSH. In addition to the traditional buffer partitions, DSH further divides the headroom into two parts: shared headroom and insurance headroom. The insurance headroom is statically reserved for each port to guarantee no congestion loss. The shared headroom is dynamically allocated as needed and shared among different ingress queues. 

Furthermore, as both shared headroom and shared buffer are dynamically shared and allocated, DSH integrates them into a single segment, collectively called shared buffer. 

Such a design brings two benefits: (1) It facilitates the implementation of DSH on switching chips, as the buffer partition is the same as the existing one on commodity switching chips. (2) It improves the buffer utilization. By integrating two kinds of buffer, both headroom and footroom share the same piece of buffer, increasing the degree of statistical multiplexing. As a result, the buffer can be more efficiently utilized. 

**缓冲区组织：** 
图7展示了DSH的缓冲区组织。除了传统的缓冲区分区外，DSH==将头部空间进一步划分为两个部分：共享头部空间和保险头部空间==。为了确保没有拥塞丢失，为**每个端口静态预留了保险头部空间**。共享头部空间根据需要进行动态分配，并在不同的入口队列之间共享。

此外，==由于共享头部空间和共享缓冲区都是动态共享和分配的，DSH将它们集成到一个单一的段中，称为**共享缓冲区（new meaning）**==。

这种设计带来了两个好处：
（1）它有助于在交换芯片上实现DSH，因为缓冲区分区与通用交换芯片上的现有分区相同。 
（2）它提高了缓冲区的利用率。通过整合两种类型的缓冲区，头部空间和脚部空间共享同一片缓冲区，增加了统计复用的程度。因此，缓冲区可以更有效地利用。

#### Buffer allocation and management：
The management of the private buffer keeps unchanged. For insurance headroom, DSH statically reserves some memory for each ingress port. Assume that there are Np ports, the insurance headroom size (denoted by Bi) is given by Bi = Np × η (4) where η is given by Eq. (1). The remaining memory is for shared buffer, which is dynamically allocated to ingress queues. DSH uses a threshold T(t) to restrict the buffer occupancy (including both shared headroom and shared footroom) for each ingress queue. Furthermore, DSH adopts DT to dynamically adjust the threshold T(t) based on the remaining buffer. DT has been widely used in commodity switching chips for decades. It can make DSH adaptive and efficient while simple to be implemented. The calculation of threshold T(t) is the same as that in existing switching chip (i.e., Eq. (2)), except that the amount of shared buffer occupancy (i.e., ωi,j(t)) should in- s clude the buffer occupancy of both shared headroom and shared footroom. Thus, DSH does not need to modify the existing hardware logic of threshold maintenance, facilitating its implementation.

**缓冲区分配和管理：** 
私有缓冲区的管理保持不变。==对于保险头部空间，DSH为每个入口端口静态保留了一些内存==。假设有 $N_p$ 个端口，保险头部空间大小（用 $B_i$ 表示）由以下公式给出： 
>$B_i = N_p * η$ (4) 
>其中，η由方程. 1）给出：
>review\[background]: 每个入口队列的头部大小（用η表示）由以下公式给出：
>$η = 2 (C · Dprop + L[MTU] ) + 3840B$

==剩余的内存用于共享缓冲区，它动态分配给入口队列==。DSH使用阈值T(t)来限制每个入口队列的缓冲区占用情况（包括 *共享头部空间 headroom* 和*共享余量空间 footroom*）。

此外，==DSH采用DT来根据剩余缓冲区动态调整阈值T(t)==。DT在通用交换芯片上已广泛使用了数十年。它使DSH具有自适应和高效的特性，同时实现简单。

阈值==T(t)的计算与现有交换芯片中的计算相同（即方程（2）），只是共享缓冲区占用量（即ωi,j(t)）应包括共享头部空间和共享脚部空间的缓冲区占用量==。因此，DSH无需修改现有的阈值维护硬件逻辑，有助于其实现。

#### Flow Control： 
DSH has two types of flow control mechanisms to guarantee against packet loss: queue-level flow control and port-level flow control. 

**流控制机制：**
DSH拥有**两种类型的流控制**机制，以确保不发生数据包丢失：*队列级流控制* 和 *端口级流控制*。
##### \[1] control way 1: 队列级流控制
The queue-level flow control is similar to the PFC mechanism. Specifically, a PFC PAUSE frame is sent to the upstream port when the amount of shared buffer occupancy for an ingress queue becomes higher than the pause threshold (denoted by $X_{qoff}$). Arriving at the upstream port, the PFC PAUSE frame will suspend the packet transmission of the corresponding traffic class. 

队列级流控制类似于PFC机制。具体来说，当入口队列的==共享缓冲区==占用量超过==暂停阈值==（由$X_{qoff}$表示）时，将向上行端口发送PFC PAUSE帧。到达上行端口时，PFC PAUSE帧将暂停相应流量类别的数据包传输。

The only difference is the setting of Xqoff threshold. With DSH, the Xqoff(t) threshold is set as Xqof f (t) = T (t) − η (5) 

唯一的区别是Xqoff阈值的设置。使用DSH时，Xqoff(t)阈值设置为:
> $X_{qoff}(t) = T(t) - \eta$

The rationale is two-fold. (1) When an ingress queue becomes congested, DSH tries to reserve enough headroom (i.e., η) for it. (2) When an ingress queue is less congested, it contributes the unused buffer to other congested queues. Specifically, the buffer occupancy of an uncongested queue is much lower than T (t). In other words, it occupies less buffer and leaves much room as free buffer, which raises the value of T (t) (recall that T (t) is proportional to the free buffer size). Accordingly, this raises the Xqoff threshold and enables other congested queues to occupy more buffer before triggering PFC messages. In summary, DSH tries to reserve enough headroom for an ingress queue if and only if it becomes congested. 

其原理有两个方面：（1）==当入口队列拥塞时，DSH试图为其保留足够的头部空间（即η）==。 （2）当入口队列拥塞较轻时，它将未使用的缓冲区贡献给其他拥塞队列。
具体来说，非拥塞队列的缓冲区占用远低于T(t)。换句话说，它占用的缓冲区较少，并留下了大量空余缓冲区，这提高了T(t)的值（回顾一下T(t)与空余缓冲区大小成正比）。

\[review]: ![[Pasted image 20231125113126.png]]
因此，这提高了$X_{qoff}$ 阈值，并使其他拥塞队列在触发PFC消息之前能够占用更多缓冲区。总之，DSH试图只有在入口队列拥塞时才为其保留足够的头部空间。

Of course, DSH cannot guarantee that every ingress queue can get η headroom when becoming congested, as the shared headroom is dynamically allocated as needed rather than stati- cally reserved beforehand. Thus, DSH also contains port-level flow control to avoid buffer overflow under any circumstances. 

==当然，DSH不能保证每个入口队列在拥塞时都能获得η头部空间==，**因为共享头部空间是根据需要动态分配的**，而不是事先静态保留的。因此，DSH还包含了端口级流控制，以在任何情况下避免缓冲区溢出。

##### \[2] control way 2: 端口级流控制
The port-level flow control is triggered when the total occupancy of shared footroom and headroom of all queues in a port becomes higher than a pause threshold (denoted by Xpoff). When triggered, the ingress port sends a port- level PAUSE frame (i.e., a PFC PAUSE frame with all pause timers of priorities set) to the upstream port. Arriving at the upstream port, the port-level PAUSE frame (i.e., a PFC PAUSE frame with all pause timers of priorities unset) will suspend the packet transmissions of all traffic classes. 

端口级流控制在以下情况下触发：当端口中所有队列的共==享脚部空间和头部空间==的总占用量超过==暂停阈值==（由$X_{poff}$表示）时。触发时，入口端口将向上行端口发送端口级PAUSE帧（即，所有优先级的暂停定时器均被设置）。到达上行端口时，端口级PAUSE帧（即，所有优先级的暂停定时器均未设置）将暂停所有流量类别的数据包传输。


The $X_{poff}(t)$ threshold is set as $X_{poff}(t)$ = $N_q$ × T(t) (6) 
$X_{poff}(t)$ 阈值设置为：
>$X_{poff}(t) = N_q \times T(t)$

The intuition is simple. DSH allocates T(t) buffer as footroom and headroom for each ingress queue. Thus, for all ingress queues in a port, the total allocated buffer is Nq ×T (t). The rationale behind the intuition is that DSH allows the ingress queues in the same port to share the allocated buffer, especially headroom. Specifically, by restricting the port-level buffer occupancy (rather than queue-level), a congested queue can occupy the headroom allocated to other queues (in the same port) if it has used up its allocated headroom. As the traffic heading for the ingress queues in the same port naturally shares the capacity of uplink, port-level buffer share is both efficient and fair. In this way, not only can DSH utilize the shared buffer more efficiently, but also the port-level flow control can be less triggered.

其直觉很简单。DSH为每个入口队列分配T(t)缓冲区作为脚部空间和头部空间。因此，对于端口中的所有入口队列，总分配的缓冲区是$Nq \times T(t)$。
这一直观背后的原理是，DSH允许同一端口中的入口队列共享分配的缓冲区，尤其是头部空间。具体来说，通过限制端口级缓冲区占用（而不是队列级），如果拥塞队列已用完其分配的头部空间，它可以占用分配给其他队列（在同一端口中）的头部空间。由于前往同一端口的入口队列的流量自然共享上行链路的容量，端口级缓冲区共享既高效又公平。通过这种方式，不仅DSH可以更有效地利用共享缓冲区，而且端口级流控制触发较少。
#### MMU workflow with DSH：
Putting all together, the workflow of DSH can be described as state transition diagrams depicted in Fig. 8 (at ingress side) and Fig. 9 (at egress side). 
将所有内容综合起来，DSH的工作流程可以用图8（在入口端）和图9（在出口端）中描绘的状态转移图来描述。

![[Pasted image 20231125202646.png]]

![[Pasted image 20231125202701.png]]

##### At ingress side, there are two queue-level states for each ingress queue:
##### 在入口端，每个入口队列有两个队列级状态：
- QON: The ingress queue is not congested. The upstream port is allowed to send packets of the corresponding traffic class to the ingress queue. Arriving packets are put into either private buffer (if $q_{i,j}≤ϕ$） or shared buffer(if $q_{i,j}>ϕ$）
- QOFF: The ingress queue is congested. The correspond- ing traffic class in the upstream port is being paused. Arriving packets are put into the shared buffer.

- QON：入口队列未拥塞。允许上行端口将相应流量类别的数据包发送到入口队列。到达的数据包被放入私有缓冲区（如果 $q_{i,j}≤ϕ$） 或者共享缓冲区（如果$q_{i,j}>ϕ$）。
- QOFF：入口队列拥塞。上行端口中相应的流量类别正在被暂停。到达的数据包被放入共享缓冲区。

Without congestion, an ingress queue is in QON state. It turns into the QOFF state when its shared buffer occupancy becomes higher than Xqoff(t). During the transition, a PFC PAUSE frame is sent to the upstream port to suspend the packet transmission of the corresponding traffic class. If the congestion is mitigated and the buffer occupancy of the ingress queue becomes lower than Xqon(t), the ingress queue turns into the QON state. During the transition, a queue-level RESUME frame (i.e., PFC PAUSE frame with zero pause duration) is sent to the upstream port to recover the packet transmission of the corresponding traffic class.

在没有拥塞的情况下，入口队列处于 QON 状态。当其共享缓冲区占用量超过 Xqoff(t) 时，它转换为 QOFF 状态。在此过程中，将向上行端口发送 PFC PAUSE 帧，以暂停==相应流量类别==的数据包传输。如果拥塞得到缓解，入口队列的缓冲区占用量降低到小于 Xqon(t)，则入口队列转为 QON 状态。在此过渡期间，将向上行端口发送队列级 RESUME 帧（即，带有零暂停持续时间的 PFC PAUSE 帧），以恢复相应流量类别的数据包传输。

##### Furthermore, for each ingress port, there are two port-level states:
##### 此外，对于每个入口端口，有两个端口级状态：
- PON: The ingress port is not congested. The upstream port is allowed to send packets if the corresponding traffic class is not paused by queue-level flow control. Arriving packets are put into either private buffer (if qi,j  φ) or shared buffer
- PON：入口端口未拥塞。允许上行端口发送数据包，==前提是相应的流量类别未被队列级流控制暂停==。到达的数据包被放入私有缓冲区（如果 qi,j≠ϕqi,j​=ϕ）或者共享缓冲区。

- POFF: The ingress port is congested. All traffic classes in the upstream port are being paused. Arriving packets are put into the insurance headroom.
- POFF：入口端口拥塞。上行端口中的所有流量类别都被暂停。到达的数据包被放入保险头部空间。

Without congestion, the port is in PON state. If the total buffer occupancy of an ingress port becomes higher than Xpoff, the ingress port turns into POFF state. During the transition, a port-level PAUSE frame is sent to the upstream port to suspend the packet transmission of all traffic classes. When the congestion of the port is mitigated and its buffer occupancy becomes lower than Xpon, the ingress port turns into PON State. During the transition, a port-level RESUME frame (i.e., PFC PAUSE frame with all pause timers unset) is sent to cease the previous (port-level) pause action. The egress-side workflow is similar except for the condi- tions/actions of state transitions, which is omitted due to space limitations.

在没有拥塞的情况下，端口处于 PON 状态。如果入口端口的总缓冲区占用量超过 XpoffX，则入口端口转为 POFF 状态。在此过程中，将向上行端口发送端口级 PAUSE 帧，以暂停==所有流量类别==的数据包传输。当端口的拥塞得到缓解，其缓冲区占用量降低到小于 Xpon 时，入口端口转为 PON 状态。
在此过渡期间，将向上行端口发送端口级 RESUME 帧（即，所有暂停定时器均未设置的 PFC PAUSE 帧），以停止先前的（端口级）暂停操作。出口端的工作流程类似，除了状态转换的条件/动作之外，由于空间限制，省略了这部分内容。

### C. DSH Analysis
>这一部分都是证明，直接白纸黑字自行操作了，此处略去！建议全部自己手证一遍！
### D. DSH Implementation
In this part, we discuss the feasibility of implementing DSH on switching chips. We show that DSH is very easy to be implemented, as it does not need to change the existing buffer architecture and only needs some moderate changes to existing flow control mechanisms.

在这部分中，我们讨论在交换芯片上实现DSH的可行性。我们展示DSH非常容易实现，因为它不需要改变现有的缓冲区架构，只需要对现有的流控制机制进行一些适度的更改。

Queue-level flow control. As shown in §II-B, existing PFC mechanism triggers PFC PAUSE messages when the queue length reaches T(t) and PFC RESUME message when the queue length reaches T(t) − δ, where δ is a configurable parameter. DSH’s queue-level flow control mechanism is the same except that the threshold to trigger a PFC PAUSE message (i.e., Xqof f ) is T (t) − η, and the threshold to trigger a PFC RESUME message (i.e., Xqon) is T(t) − η − δq. Thus, we only need to change the conditions of generating PFC message. Specifically, the switching chip needs two extra subtractors. One is for calculating the Xqoff threshold, whose inputs are T(t) and η. The other is for calculating the Xqon threshold, whose inputs are Xqof f and δq

**队列级流控制**。如第II-B节所示，现有的PFC机制在队列长度达到T(t)时触发PFC PAUSE消息，在队列长度达到T(t)−δ时触发PFC RESUME消息，其中δ是可配置的参数。DSH的队列级流控制机制相同，只是触发PFC PAUSE消息的阈值（即，Xqoff​)为T(t)−η，而触发PFC RESUME消息的阈值（即，Xqon​)为T(t)−η−δq​。因此，我们只需要改变生成PFC消息的条件。具体而言，交换芯片需要两个额外的减法器。一个用于计算Xqoff​的阈值，其输入为T(t)和η。另一个用于计算Xqon​的阈值，其输入为Xqoff​和δq​。

Port-level flow control. Lots of commodity switching chips have already supported port-level flow control based on port- wise buffer occupancy [29], [35]. Thus, to achieve port- level flow control, we only need to change the conditions of generating port-level PAUSE/RESUME messages. The pause threshold (i.e., Xpof f ) is Nq · T (t), where Nq is the number of queues per port. The switching chip needs a multiplier, whose inputs are Nq and T(t), to calculate the pause threshold. In particular, if N is a power of two, only a shift register is needed for the calculation. The resume threshold (i.e., Xpon) is Xpff - δp, where δp is a configurable parameter. The switching chip needs a subtractor, whose inputs are Xpoff and δp, to calculate the resume threshold.

**端口级流控制**。许多通用交换芯片已经支持基于端口缓冲区占用的端口级流控制[29]，[35]。因此，为了实现端口级流控制，我们只需要改变生成端口级PAUSE/RESUME消息的条件。暂停阈值（即，Xpoff​)为 Nq​⋅T(t)，其中Nq​是每个端口的队列数。交换芯片需要一个乘法器，其输入为Nq​和T(t)，以计算暂停阈值。特别地，如果Nq是2的幂次方，则只需要一个移位寄存器进行计算。恢复阈值（即，Xpon​)为Xpoff​−δp​，其中δp​是可配置的参数。交换芯片需要一个减法器，其输入为Xpoff​和δp​，以计算恢复阈值。

Consolidating two kinds of flow controls. For the down- stream port, two kinds of flow control work independently, and thus no further modifications are required to consolidate them. For the upstream port, we only need to change the condition of suspending packet transmissions. Specifically, each egress queue suspends packet transmissions if the egress queue is in QOFF state or the egress port is in POFF state. To realize this, the switching chip needs to maintain a queue-level state per egress queue and a port-level state per egress port as Fig. 9 shows. Then the pause condition can be easily generated by an OR gate.

**整合两种流控制**。对于**下行端口**，**两种流控制工作独立**，因此无需进行进一步的修改来整合它们。对于**上行端口**，我们只需要**改变暂停数据包传输的条件**。具体来说，如果*出口队列处于 QOFF 状态* 或者 *出口端口处于POFF状态*，每个出口队列都会暂停数据包传输。为了实现这一点，交换芯片需要维护==每个出口队列的队列级状态==和==每个出口端口的端口级状态==，如图9所示。然后，通过一个 *OR门* 可以轻松生成暂停条件。
![[Pasted image 20231125202701.png]]
review：OR门
>OR门是数字电路中的一种逻辑门，用于执行逻辑运算。对于两个输入（通常被称为A和B），OR门的输出为：
Q=A 或 BQ=A 或 B
也就是说，如果输入A或B至少有一个是1（高电平），那么输出Q就是1。只有当A和B都是0时，输出Q才是0。这种逻辑运算对应于“或”关系，因此它的输出是输入中任何一个为真（1）即为真（1）。

Overall resource increments to the switch ASIC. Based on the above analysis, we argue that DSH brings acceptable resource increments to switch ASIC due to the following rea- sons. (1) DSH does not require additional counters/registers. DSH only requires the buffer occupancy of each queue/port, which is already available in existing switch ASIC. (2) DSH does not touch the memory allocation and management mech- anisms. DSH’s physical memory architecture (Fig. 7) is the same as the existing one in commodity switches (Fig. 2). Furthermore, DSH does not modify the the memory allocation and management mechanisms of footroom buffer, while the al- location of headroom buffer can be realized by additional flow control mechanisms. Thus, implementing DSH does not need to modify the existing memory allocation and management mechanisms. (3) Simple and cheap comparison/arithmetic op- erations are required to realize the conditions of triggering PFC messages. DSH can be implemented by modifying the conditions of triggering PFC messages, while all conditions are based on comparison between buffer occupancy (and its derivatives) and thresholds. Thus, DSH only requires compar- ison and simple arithmetic operations (i.e., subtraction).

**总体而言，对交换芯片的资源增加**。基于以上分析，我们认为DSH对交换芯片带来了可接受的资源增加，原因如下：
1. DSH不需要额外的计数器/寄存器。DSH只需要每个队列/端口的缓冲区占用情况，而这在现有的交换芯片中已经是可用的。
2. DSH不涉及内存分配和管理机制。DSH的物理内存架构（图7）与通用交换机中的现有架构（图2）相同。此外，DSH不修改脚部缓冲区的内存分配和管理机制，而头部缓冲区的分配可以通过额外的流控制机制实现。因此，实施DSH不需要修改现有的内存分配和管理机制。
3. 实现触发PFC消息的条件只需要简单且廉价的比较/算术操作。DSH可以通过修改触发PFC消息的条件来实现，而所有条件都基于缓冲区占用（及其派生值）与阈值之间的比较。因此，DSH只需要比较和简单的算术操作（即减法）。