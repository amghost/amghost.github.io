# broker注册
broker在zookeeper中创建临时节点`/brokers/ids/<broker_id>`完成注册。
- 如果临时节点已存在，注册失败
- 如果出现broker停机、网络分区、长时间GC停顿，broker从zookeeper上断开连接，此时临时节点自动从zookeeper中移除
- 其他brokers通过监听`/brokers/ids`获知broker的注册和退出事件

# 控制器controller
controller是一种角色，有其中一个broker来担任。除了具有一般broker的功能外，还负责分区首领的选举。

- 集群第一个加入的broker会尝试添加一个 `/controller` 的临时节点让自己成为控制器
- 其他broker监听该临时节点获知controller的变更通知
- 当controller关闭或者与zookeeper断开连接，临时节点被移除，其他brokers通过watch对象收到通知，并会尝试让自己成为controller
- 每个新选出的控制器通过zookeeper的条件递增操作获得一个全新的、数值更大的controller epoch。其他broker在知道当前controller epoch之后，如果收到由控制器发出的较小的epoch（脑裂导致），就会忽略它们

# 分区复制
分区副本（replica）分为 **首领副本** 和 **跟随者副本**。 只有首领副本会处理生产者和消费者的请求，跟随者只负责同步首领副本的数据

