# Conclusion

>In datacenter networks, PFC-enabled switches need to re-serve some buffer as headroom to avoid buffer overflow. How-ever, with the growing link speed, the buffer becomes increas-ingly inadequate, and the headroom occupies a considerable fraction of buffer, significantly squeezing the buffer space for accommodating normal traffic. As a result, PFC messages can be frequently generated, bringing about serious performance impairments. In this paper, we argue that the current static and queue-independent headroom allocation scheme is inherently inefficient. We propose Dynamic and Shared Headroom (DSH) allocation scheme, which dynamically allocates headroom to congested queues and enables allocated headroom to be shared among different queues. Extensive simulations show that DSH can significantly reduce the PFC messages and improve the network performance.

在数据中心网络中，启用PFC的交换机需要重新保留一些缓冲区作为头部空间，以避免缓冲区溢出。
然而，随着链路速度的增长，缓冲区变得越来越不足，头部空间占据了缓冲区的相当大一部分，显著挤压了用于容纳正常流量的缓冲区空间。
结果，PFC消息可能会频繁生成，导致严重的性能损害。

在本文中，我们认为当前的静态和与队列无关的头部空间分配方案本质上是低效的。
我们提出了一种动态和共享的头部空间（DSH）分配方案，该方案动态地为拥塞的队列分配头部空间，并使分配的头部空间可以在不同队列之间共享。

大量的仿真结果表明，DSH能够显著减少PFC消息并改善网络性能。