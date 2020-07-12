# DaemonSet

daemon, 就是用来部署守护进程的，DaemonSet用于在每个kubernetes节点中将守护进程的副本作为后台进程运行，说白了就是在每个节点部署一个Pod副本，当节点加入到kubernetes集群中，Pod会被调度到该节点上运行，当节点从集群中被移除后，该节点上的这个Pod也会被移除。当然，如果我们删除DaemonSet，所有和这个对象相关的Pods都会被删除。


在哪种情况下我们会需要用到这种业务场景呢？

- 集群存储守护程序，如glusterd，ceph要部署在每个节点上以提供持久化存储
- 节点监控守护进程，如prometheus监控集群，可以在每个节点上运行一个node-exporter进程来收集监控节点的信息
- 日志收集守护进程，如fluentd或logstash


这里需要特别说明的一个就是关于DaemonSet运行的Pod的调度问题，正常情况下，Pod运行在哪个节点上是由kubernetes的调度器策略来决定的。然而，由DaemonSet控制器创建的Pod实际上提前已经确定了在哪个节点上（Pod创建时指定了.spec.nodeName）, 所以：
  * DaemonSet并不关心一个节点的unschedulable字段
  * DaemonSet可以创建Pod，即使调度器还没有启动，这点非常重要


下面使用一个示例来演示下，在每个节点上部署一个nginx pod：（nginx-ds.yaml）