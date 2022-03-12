## Hbase 的重要工作原理

**- region 分配**
1. 任何时刻，一个region 只能分配给一个region server
2. master 记录了当前哪些可用的region server ，以及当前哪些region分配给了哪些region server,哪些region server 还未分配或者还空闲。当需要分配新的region ，这个时候master 就会找一个region server 上有可用空间的，并给这个server 发送一个装载请求，把这个region 分配给这个region server。region server 接到请求后就开始对此region 提供服务。


## 注：通过ZooKeeper 的wacth 机制和临时节点来感知从节点得上下线
**- region server 上线**

1. master 使用Zookeeper 来跟踪region server 状态
2. 当某个region serve 启动时：
 首先在zookeeper 上得server 目录下建立代表自己得znode节点（临时节点-会话机制）
； 由于master 订阅了server 目录上得变更信息，当server目录下得文件出现新增或者删除操作时，master可以得到自zookeeper 得实时通知；
一旦region server上线，master 能马上得到消息

**- region server 下线**

1. 当region server 下线时，它和zookeeper 的会话会断开，Zookeeper而自动释放代表这台server的文件上得独占锁
2. master 就可以确定：region server 和zookeeper 之间的网络断开了；region server 挂了
3. 无论哪种情况，region server 都无法继续为它得region 提供服务了，此时master 会删除server 目录下代表这台region server 的znode 数据，并将这台region server 的region 分配给其他还活着得节点

## 注：通过ZooKeeper 的wacth 机制和临时节点和权限机制来感知主节点得上线
**- master 上线** ==（抢锁机制）==
1. 从zookeeper 上获取唯一个代表active master 的锁，用来阻止其他但master 成为master(即多台master 的情况下)
2. 一般hbase集群中总是有一个master在提供服务，还有一个以上得master在standby 中
3. 扫描zookeeper上得server 父节点，获得当前可用得region server列表
4. 和每个region server 通信，获得当前已分配得region和region server的对应关系
5. 扫描.meta 、region 的集合 ，计算得到当前还未分配得region，将他们防入待分配但region 列表

**- master 下线**

1. 由于==master 只维护表和region 的元数据==，而不参与表数据io 的过程，master 下线仅导致所有的元数据的修改被冻结包括：无法创建删除表；无法修改表得schema;无法进行region server的负载均衡 ；无法处理region server上线；无法进行region 的合并；唯一例外得是region 的split可以正常进行，因为只有region server 参与；==表数据的读写还可以正常进行==
2. 因此master 下线短时间内对整个hbase集群没有影响
3. 从上线过程可以看到，master 保存得信息全部是冗余信息(都可以从系统其他地方收集或者计算出去)

## Hbase 批量装载 --bulk load
1. 先生成Hfile 文件
2. 在通过load ️阶段加入hbase表

## Hbase 的协处理器 （Compressor）

## Hbase 的调优说明


## Hbase 的数据结构

