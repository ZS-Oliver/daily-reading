# 租约
- 集群重的 node 需要对一些资源独占
  - 为什么不用锁
    - 持有锁的 node 可能会 crash
    - 或者是短暂的网络失联，进程暂停
    - 这些都导致锁不能被正常释放
    - 分布式锁实现还是有很多坑，有兴趣可以看这篇 [how to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
      - 分布式中总会牵扯到 liveness 和 safety 的权衡问题。

## 租约的实现
- 租约是有时限，并且会失效，这就保证了不会出现无法释放的情况
- 租约还可以 renew ，如果想延长操作
- leader 负责跟踪租约的过期，并和所有 node 达成共识
- 看了一会，发现这个租约就是 raft 论文里面如何保证读一致性的方案
- 如何避免重复的租约
  - 在添加租约时检查
- lease 怎么刷新
  - 多发几次以防止网络问题
  - 避免太多，最好在租约期限过半时发送，并且有一定间隔
- 怎么监听某个 node 是否存活
  - 每个 node 都有一个带有自身标识的 lease，并且不断 fresh ，若果过期了，就可以通知感兴趣的服务
  - 像是 zookeeper 的临时节点
- leader 宕机了咋办
  - 其实只要保证同时不会有两个 leader ，那就不会有影响
  - 因为只有 leader 能续约
  
## 小结
- 希望哪天手写 raft 的时候能用到这篇文章，感觉现在都是思考太多，动手太少。
