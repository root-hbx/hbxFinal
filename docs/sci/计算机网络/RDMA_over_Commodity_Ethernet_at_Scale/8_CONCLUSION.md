>In this paper, we have presented our practices and experiences in deploying RoCEv2 safely at large-scale in Microsoft data centers. Our practices include the introducing of DSCP-based PFC which scales RoCEv2 from layer-2 VLAN to layer-3 IP and the step-by-step onboarding and deployment procedure. Our experiences include the discoveries and resolutions of the RDMA transport livelock, the RDMA deadlock, the NIC PFC storm and the slow-receiver symptom. With the RDMA management and monitoring in place, some of our highlyreliable, latency-sensitive services have been running RDMA for over one and half years.

在本文中，我们介绍了在 Microsoft 数据中心大规模安全部署 RoCEv2 的实践和经验。
我们的实践包括引入基于 DSCP 的 PFC，将 RoCEv2 从第 2 层 VLAN 扩展到第 3 层 IP，以及逐步的加入和部署过程。
我们的经验包括发现并解决 RDMA 传输活锁、RDMA 死锁、NIC PFC 风暴和缓慢接收器症状。
随着 RDMA 管理和监控到位，我们的一些高度可靠、延迟敏感的服务已经运行 RDMA 超过一年半了。

### 8.1 Future Work
>There are several directions for our next steps. The hop-by-hop distance for PFC is limited to 300 meters. Hence RoCEv2 works only for servers under the same Spine switch layer. To this end, RoCEv2 is not as generic as TCP. We need new ideas on how to extend RDMA for inter-DC communications. Our measurement showed ECMP achieves only 60% network utilization. For TCP in best-effort networks, there are MPTCP [29] and per-packet routing [10] for better network utilization. How to make these designs work for RDMA in the lossless network context will be an interesting challenge. The deadlock we have discovered in this paper reminds us that deadlock in data centers may be worthy of more systematic study. Even though the up-down routing in Clos network can prevent deadlock, designs like F10 [23] may break the assumption by introducing local rerouting. Many other network topologies [20] do not even have the up-down routing property. How can deadlocks be avoided in those designs? Last but not least, we have shown that RDMA provides low latency and high throughput by eliminating OS kernel packet processing overhead and by relying on a lossless network. A lossless network, however, does not guarantee low latency. When network congestions occur, queues build up and PFC pause frames may still be generated. Both queues and PFC pause frames increase network latency. How to achieve low network latency and high network throughput at the same time for RDMA is still an open problem.

我们的下一步有几个方向：
1. 由于PFC 的逐跳距离限制为 300 米，所以RoCEv2 仅适用于同一 Spine 交换机层下的服务器。为此，RoCEv2 不像 TCP 那样通用。我们需要关于如何扩展 RDMA 以实现 DC 间通信的新想法。
2. 我们的测量显示 ECMP 仅实现 60% 的网络利用率。对于尽力而为网络中的 TCP，有 MPTCP [29] 和每数据包路由 [10]，以实现更好的网络利用率。如何使这些设计在无损网络环境下适用于 RDMA 将是一个有趣的挑战。
3. 我们在本文中发现的死锁提醒我们，数据中心的死锁可能值得更系统的研究。尽管 Clos 网络中的上下路由可以防止死锁，但像 ==F10 [23] 这样的设计==可能会通过引入本地重新路由来打破这一假设。
4. 许多其他网络拓扑[20]，它们甚至不具有上下路由属性。在这些设计中如何避免死锁？
5. Last but not least，我们已经证明:  RDMA 通过消除操作系统内核数据包处理开销并依赖无损网络来提供低延迟和高吞吐量。然而，无损网络并不能保证低延迟。当网络拥塞发生时，队列堆积并且仍然可能生成PFC暂停帧。队列和 PFC 暂停帧都会增加网络延迟。 RDMA如何同时实现低网络延迟和高网络吞吐量仍然是一个悬而未决的问题。