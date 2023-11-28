# Background

In this section, we introduce the background of PFC and switch buffer
### A. Priority-based Flow Control
Ethernet-based datacenter networks rely on Priority-based Flow Control (PFC) [20] to guarantee lossless packet forwarding. PFC is a hop-by-hop flow control mechanism (as shown in Fig. 1). In a PFC-enabled switch, once the length of an ingress queue exceeds a preset threshold ( i.e., $Xoff$ ), the switch sends a PAUSE frame to the upstream device. Receiving the PAUSE frame, the upstream device suspends the packet transmission for some duration specified by the PAUSE frame. When the ingress queue length falls below another threshold ( i.e., $Xon$ ), the device sends a PAUSE frame with zero duration (we refer to it as RESUME frame in this paper) to the upstream device, which recovers the packet transmission.

以以太网为基础的数据中心网络依赖于基于优先级的流控制（PFC）[20]来保证无丢包的数据转发。PFC是一种逐跳的流控制机制（如图1所示）。在启用了PFC的交换机中，一旦入口队列的长度超过预设阈值（即$Xoff$），交换机就会向上游设备发送一个==**暂停帧（PAUSE frame）**== 。接收到暂停帧后，上游设备会暂停一定时间，该时间由暂停帧中指定。当入口队列长度降到另一个阈值以下（即$Xon$）时，设备会向上游设备发送一个暂停帧，但该帧的持续时间为零（我们在本文中称之为==**恢复帧，RESUME frame**==），从而恢复数据传输。

To prevent packet dropping, $Xoff$ should be set conservatively. This is because it takes time for the PAUSE frame to arrive at the upstream device and take effect. To prevent buffer overflow, enough buffer beyond $Xoff$ should be reserved to accommodate in-flight packets during this time. The reserved buffer beyond $Xoff$ is called **buffer headroom**.

为防止数据包丢失，$Xoff$ 应该被谨慎设置。这是因为PAUSE帧需要时间传输到上游设备并生效。为了防止缓冲区溢出，在$Xoff$ 之外需要保留足够的缓冲区来容纳在此期间的在途数据包。在$Xoff$ 之外的保留缓冲区被称为**缓冲头部余量**。

In the PFC standard [20], traffic can be classified into 8 priority classes. Each priority class is mapped to a separate queue and packets belonging to different priority classes are placed into different queues. The PFC messages carry the priority information and can only pause/resume a specific traffic class.

在PFC标准[20]中，流量可以被分类为8个优先级类别。每个优先级类别被映射到一个单独的队列，不同优先级类别的数据包被放置到不同的队列中。PFC消息携带优先级信息，只能暂停/恢复特定流量类别。
![[Pasted image 20231125105256.png]]
### B. Buffer Architecture in PFC-enabled Switches

On a switching chip, the packets waiting to be transmitted are stored in a packet buffer. Today’s commodity high-speed switching chips usually employ on-chip shared memory for high-bandwidth and low-latency packet access [3], [13], [21]– [28].

在一个**交换芯片**上，等待传输的数据包存储在**数据包缓冲区**中。如今的通用高速交换芯片通常采用芯片内共享内存来实现高带宽和低延迟的数据包访问[3]，[13]，[21]–[28]。

Basically, in a PFC-enabled switch, the buffer is partitioned into two pools [25], [29], [30] (as shown in Fig. 2). One pool is dedicated to lossless traffic that relies on PFC to avoid congestion loss. The other pool is dedicated to lossy traffic that is allowed to be dropped due to buffer overflow. Buffer is hard partitioned in such way to ensure performance isolation between two kinds of traffic. In this paper, we mainly focus on the lossless pool.

基本上，在启用PFC的交换机中，缓冲区被分为两个池[25]，[29]，[30]（如图2所示）。一个池专用于依赖PFC避免拥塞丢包的**无丢包流量**。另一个池专用于**允许由于缓冲区溢出而丢弃的有丢包流量**。缓冲区被**硬性划分，以确保两种流量之间的性能隔离**。在==本文中，我们主要关注无丢包池==。

In the lossless pool, the buffer is further partitioned into three segments:
- Private buffer: buffer space reserved for each queue, which guarantees each queue’s minimum buffer resource.
- Shared buffer: buffer space shared among all queues.   
- Headroom buffer: buffer space reserved for each queue, which absorbs in-flight packets after sending PAUSE
 frames.

在无丢包池中，缓冲区进一步分为三个部分：
- 私有缓冲区：为每个队列保留的缓冲区空间，以保证每个队列的最小缓冲资源。
- 共享缓冲区：在所有队列之间共享的缓冲区空间。
- 头部缓冲区：为每个队列保留的缓冲区空间，用于在发送PAUSE帧后吸收在途数据包。
![[Pasted image 20231125105318.png]]

### C. Buffer Allocation and Management in the Lossless Pool
The switching chip utilizes a Memory Management Unit (MMU) to allocate the buffer to arriving packets. For lossless traffic, MMU manages the buffer from the ingress perspective [29]–[31] (i.e., allocating buffer to ingress ports/queues). The sizes of private buffer and headroom buffer are explicitly configured. The remaining buffer serves as shared buffer.

交换芯片利用**内存管理单元（Memory Management Unit，MMU）** 来为到达的数据包分配缓冲区。对于无丢包流量，MMU从入口的角度进行缓冲区管理[29]–[31]（即为入口端口/队列分配缓冲区）。私有缓冲区和头部缓冲区的大小是显式配置的，而剩余的缓冲区用作共享缓冲区。

**Private buffer：**. There is no explicit rule specifying how to configure the private buffer size. Nevertheless, the amount of private buffer is relatively small (e.g., 16% in Arista 7050X3 switches [26]).

**私有缓冲区：** 没有明确的规则规定如何配置私有缓冲区的大小。然而，私有缓冲区的量相对较小（例如，在Arista 7050X3交换机中为16%）。

**Headroom buffer：** Different from the private buffer, the size of the headroom buffer should be carefully configured to prevent packet loss. This is because it takes some delay for a PAUSE frame to take effect, and the MMU needs to reserve enough headroom to absorb the arriving traffic during this delay. According to [1], [3], [32], [33], the headroom size for each ingress queue (denoted by η) is given by

**头部缓冲区：** 与私有缓冲区不同，*头部缓冲区的大小需要仔细配置以防止数据包丢失*。这是因为==PAUSE帧需要一些延迟才能生效，而MMU需要保留足够的头部余量来吸收在此延迟期间到达的流量==。根据[1]，[3]，[32]，[33]，每个入口队列的头部大小（用η表示）由以下公式给出：

> $η = 2 (C · Dprop + L[MTU] ) + 3840B$

where C is the capacity of the upstream link, $Dprop$ is the propagation delay of the upstream link, and $L[MTU]$ is the length of an MTU-sized packet. The rationale of such a setting is as follows. The delay for PFC pause to take effect comprises the following five parts:

ps：在计算机网络中，**MTU**代表**最大传输单元（Maximum Transmission Unit）**。MTU是指在通信中能够一次性传输的数据包的最大尺寸。通常，MTU以字节为单位表示，它**决定了网络中能够传输的最大数据包的大小**。

其中，C是上行链路的容量，$D_{prop}$是上行链路的传播延迟，$L[MTU]$是一个以MTU为大小的数据包的长度。这种设置的基本原理如下。PFC暂停生效所需的延迟包括以下五个部分：

1. Waiting delay: A port may be busy transmitting another packet when a PAUSE frame is generated. The PAUSE frame needs to wait for the transmission to be finished. In the worst case, the port just begins to transmit the first bit of an MTU-sized packet, and thus the PAUSE frame needs to wait for LMTU/C time.
2. Propagation delay (of PAUSE frame): It takes $Dprop$ time for the PAUSE frame to arrive at the upstream device. $Dprop$ depends on the cable length and propagation speed of signals. In data centers, the distance between two connected switches can be as large as 300 meters [3]. For single-mode fibers, the speed of light is 65% of that in a vacuum. As a result, the propagation delay is ∼1.5μs.
3. Processing delay: It takes some time for the switch to process the PAUSE frame and stop the transmission. The PFC definition has capped this time to 3840B/C [32].
4. Response delay: When the upstream port decides to execute the pause action, it might be sending another packet. In the worst case, the switch just begins to transmit the first bit of an MTU-sized packet. Thus, it takes LMTU/C for the pause action to truly take effect.
5. Propagation delay (of the last packet): When the upstream device stops sending packets, there are still some in-flight packets on the link, which should also be absorbed by the headroom. It takes another Dprop time for the last sent packet to arrive at the downstream switch.
Combining the above five parts results in Eq. (1).

1. 等待延迟：当生成一个PAUSE帧时，端口可能正在传输另一个数据包。PAUSE帧需要等待传输完成。在最坏的情况下，端口刚开始传输一个以MTU为大小的数据包的第一个比特，因此PAUSE帧需要等待LMTU/C的时间。✅
2. 传播延迟（PAUSE帧的）：PAUSE帧到达上游设备需要$Dprop$的时间。$Dprop$取决于电缆长度和信号传播速度✅。在数据中心，两个相连交换机之间的距离可能长达300米[3]。对于单模光纤，光速是真空中的65%。因此，传播延迟约为1.5微秒。
3. 处理延迟：交换机处理PAUSE帧并停止传输需要一些时间。PFC定义将此时间限制为3840B/C✅ [32]。
4. 响应延迟：当上游端口决定执行暂停操作时，它可能正在发送另一个数据包。在最坏的情况下，交换机刚开始传输一个以MTU为大小的数据包的第一个比特。因此，执行暂停操作真正生效需要LMTU/C的时间。✅
5. 传播延迟（最后一个数据包的）：当上游设备停止发送数据包时，链路上仍然存在一些在途数据包，这些数据包也应该被头部余量吸收。最后一个发送的数据包到达下游交换机需要另外的Dprop时间。✅
*我的计算：ß = 容量 = 总用时 x c(光速，传输速度)*
结合这五个方面，可以得出上述公式(1)

**Shared buffer：** The shared buffer is available to all queues. MMU utilizes a buffer management scheme to ensure fair and efficient allocation of the shared buffer among all queues. Among various buffer management schemes, Dynamic Threshold (DT) is the most common one on commodity switching chips [3], [4], [21]–[23], [26], [29], [34]–[36]. DT uses a threshold to restrict the length of each queue. The threshold is dynamically adjusted according to the remainingbuffer size. Specifically, let T(t) denote the threshold at time
t, $w^{(i,j)}{(t)}$ denote the amount of shared buffer occupation for s queue j in port i at time t, and Bs denote the buffer size of the shared segment. The threshold is given by：

**共享缓冲区：** 共享缓冲区对所有队列都是可用的。MMU利用一种缓冲区管理方案来确保在**所有队列**之间**公平且高效**地分配共享缓冲区。在各种缓冲区管理方案中，**动态阈值（Dynamic Threshold，DT）** 是通用交换芯片上最常见的方案之一[3]，[4]，[21]–[23]，[26]，[29]，[34]–[36]。*DT使用阈值来限制每个队列的长度*。该阈值**根据剩余缓冲区大小进行动态调整**。具体而言，让==T(t)表示时间t时的阈值==，==$w^{(i,j)}{(t)}$表示时间t时端口i中队列j的共享缓冲区占用量==，$Bs$表示共享段的total缓冲区大小。阈值由以下公式给出：

>![[Pasted image 20231125113126.png]]

where α is a control parameter. The rationale behind DT is as follows. When the network is less congested, the amount of buffer occupancy is low and thus the remaining buffer size is high. DT adjusts the threshold to a higher value, which enables each queue to occupy more buffer, making the buffer efficiently used. On the contrary, when the network is more congested, the amount of buffer occupancy is high and thus the remaining buffer size is low. DT adjusts the threshold to a lower value, which restricts the buffer occupations of each queue, making the buffer fairly shared among different queues.

其中，α是一个控制参数。==DT背后的原理==如下。当网络拥塞较少时，缓冲区占用量较低，因此剩余缓冲区大小较大。DT将阈值调整到较高的值，使每个队列能够占用更多缓冲区，从而有效利用缓冲区。相反，当网络拥塞较多时，缓冲区占用量较高，因此剩余缓冲区大小较小。DT将阈值调整到较低的值，限制每个队列的缓冲区占用，使缓冲区在不同队列之间公平共享。[真的妙！！！！！！]

**MMU workflow with PFC enabled：** With PFC enabled, MMU monitors the ingress queue lengths (denoted by qi,j(t)) and decides where to place each arriving packet. Further- more, MMU generates PFC PAUSE messages to upstream devices based on the amount of shared buffer occupancy and Xof f /Xon thresholds1 . The PFC mechanism can be described as the state transition diagram in Fig.3.

**启用PFC的MMU工作流程：** 启用PFC后，MMU监视入口队列长度（用$q_{i,j}(t)$表示），并决定将每个到达的数据包放置在何处。此外，MMU根据共享缓冲区占用量和$X_{off}/X_{on}$ 阈值生成PFC PAUSE消息发送给上游设备。PFC机制可以描述为图32中的状态转换图。
![[Pasted image 20231125123217.png]]

1. With private buffer, the Xoff / Xon thresholds are compared with the amount of shared buffer occupancy, rather than queue length.
2. Note that the PFC standard uses pause timers to suspend the traffic transmission rather than ON/OFF states. Nonetheless, it is logically identical to the ON/OFF signals as the pause duration specified by a PAUSE frame is usually longer than the duration for the queue length to fall bellow the Xon threshold. Thus, we use ON/OFF states to describe the PFC mechanism for the ease of understanding.

1. **对于==私有缓冲区，Xoff/Xon阈值与共享缓冲区占用量进行比较==，而不是队列长度。**
2. **请注意，PFC标准==使用暂停计时器来暂停流量传输，而不是ON/OFF状态==。尽管如此，从逻辑上讲，它与ON/OFF信号是相同的，因为由PAUSE帧指定的暂停持续时间通常比队列长度下降到Xon阈值以下所需的时间长。因此，我们==使用ON/OFF状态来描述PFC机制，以便更容易理解==。**

Specifically, for each arriving packet, there are four cases:
1. $qi,j​(t)<ϕ$ : (where ϕϕ is the amount of private buffer reserved for each ingress queue): MMU puts the packet into the private buffer.
2. $ϕ≤qi,j(t)<ϕ+T(t)ϕ≤qi,j​(t)<ϕ+T(t)$ : MMU puts the packet into the shared buffer of port i,ji,j. Furthermore, if the ingress queue is in OFF state and ωi,j(t)<Xon, MMU triggers a RESUME frame (i.e., PFC PAUSE frame with zero pause duration) to the upstream device and makes the ingress queue turn into ON state.
3. $ϕ+T(t)≤qi,j(t)<ϕ+T(t)+ηϕ+T(t)≤qi,j​(t)<ϕ+T(t)+η$ :  MMU puts the packet into the headroom buffer. Furthermore, if the ingress queue is in ON state, MMU triggers a PAUSE frame to the upstream device and makes the ingress queue turn into OFF state.
4. $qi,j(t)≥ϕ+T(t)+ηqi,j​(t)≥ϕ+T(t)+η$ :  MMU drops the packet.

具体而言，对于每个到达的数据包，存在以下四种情况：
1. $q_{i,j}(t) < \phi$（其中$\phi$是为每个入口队列保留的私有缓冲区量）：MMU将数据包放入私有缓冲区。✅
2. $\phi \leq q_{i,j}(t) < \phi+T(t)$：MMU将数据包放入共享缓冲区。此外，如果入口队列处于OFF状态且$\omega_{i,j}(t) < Xon$，MMU触发一个RESUME帧（即，带有零暂停持续时间的PFC PAUSE帧）发送给上游设备，并使得入口队列进入ON状态。
3. $\phi+T(t) \leq q_{i,j}(t) < \phi+T(t)+\eta$：MMU将数据包放入头部缓冲区。此外，如果入口队列处于ON状态，MMU触发一个PAUSE帧发送给上游设备，并使得入口队列进入OFF状态。
4. $q_{i,j}(t) \geq \phi + T(t) + \eta$：MMU丢弃数据包。