# zookeeper
- 用途
  - 分布式系统协调系统

- ZAB（Zookeeper Atomic Broadcast）原子消息广播协议
  - 发现：选举Leader过程
  - 同步：Learner从Leader中同步最新的数据
  - 广播：接收客户端的事务请求，并进行消息广播，实现数据在集群节点的副本存储

- Paxos
  - 基于消息传递的一致性算法
  - 投票传递
  - 影响因素
    - 数据新旧程度：只有最新数据的节点才有机会成为Leader，zxid越大越新
    - myid：节点的数据zxid相同时，数字最大的就会被选为Leader，如果Leader确定，新加入的节点不会影响原来的集群。
    - 投票数过半

- 一致性处理过程
  - 事件注册
  - 监听事件
  - 回调函数

- 角色
  - leader 负责投票的发起喝决议，更新系统状态
  - learner
    - follower 用于接收客户端请求并向客户端返回结果，在选主中参与投票
    - observer 仅用于扩展系统，提高读取速度，不参与投票
    
- 服务状态（角色）
  - LOOKING 搜寻集群Leader节点
  - LEADING 领导者状态
  - FOLLOWING 跟随者状态
  - OBSERVING 观察者状态
  
- 节点状态 znode
  - PERSISTENT 持久化节点
  - PERSISTENT_SEQUENTIAL 持久化顺序编号节点
  - EPHEMERAL 临时目录节点（客户端在一个会话结束后，【心跳检测到结束后】删除该znode）
  - EPHEMERAL_SEQUENTIAL 临时顺序编号目录节点（客户端在一个会话结束后，【心跳检测到结束后】删除该znode）

- CAP
  - Consistency 一致性：数据在多个副本间保持一致
  - Availability 可用性：服务一直处于可用状态，用户请求在有限时间内返回结果
  - Partition tolerance 分区容忍性：脑裂发生时，集群能够提供一定的可用性和一致性
    - 在一个分布式系统中，最多只能满足其中两项。分布式系统必须满足分区容忍性，那么其它两项就可根据具体需求作取舍。

- 配置项
  - myid 范围1-255
  - 2888 follower与leader之间通信和数据同步的端口
  - 3888 选举过程中投票通信端口
  - ticktime 时钟，默认2000毫秒
  - initLimit leader等待follower启动和数据同步的集群初始化时钟倍数，默认10*2秒
  - syncLimit leader与follower之间的心跳检测的最大延迟时钟倍数，默认5*2秒

- 集群安装
  - 统一操作（为避免重复，以下操作可拷贝快照）
    - 安装 JVM
    - 下载zookeeper-3.5.5-bin.tar.gz wget https://archive.apache.org/dist/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz
    - 解压 tar -zxf apache-zookeeper-3.5.5.tar.gz
    - 环境变量 /etc/profile export ZK_HOME=/usr/local/application/apache-zookeeper-3.5.5
    - cp conf/zoo_sample.cfg conf/zoo.cfg
    - mkdir -p /usr/local/application/zk/data
    - dataDir=/usr/local/application/zk/data
    - 增加配置项
      ```
      server.1=kafka0:2888:3888
      server.2=kafka1:2888:3888
      server.3=kafka2:2888:3888
      ```
    - 关闭防火墙 systemctl stop firewalld && systemctl disable firewalld
   - 独立配置
     - echo 1 > /usr/local/application/zk/data/myid
     - echo 2 > /usr/local/application/zk/data/myid
     - echo 3 > /usr/local/application/zk/data/myid
   - 脚本命令
     - 开启服务 zkServer.sh start
     - 服务状态 zkServer.sh status
     - 使用客户端 zkCli.sh
       - 创建节点 create /test abc
       - 查看节点 ls /
       - 获取节点 get /test
       - 修改节点 set /test def
       - 删除节点 rmr /test
       - 节点详情 stat /test
         ```
          cZxid = 0x100000002 # 节点创建事务ID
          ctime = Sun Oct 06 02:21:45 CST 2019
          mZxid = 0x100000002 # 节点修改事务ID
          mtime = Sun Oct 06 02:21:45 CST 2019
          pZxid = 0x100000004 # 子节点创建/删除事务ID
          cversion = 1
          dataVersion = 0
          aclVersion = 0
          ephemeralOwner = 0x0 # 临时节点标记，在一次会话结束后失效
          dataLength = 3
          numChildren = 1
         ```
