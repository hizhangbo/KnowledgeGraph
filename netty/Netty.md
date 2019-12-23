# Netty
- 三种I/O模型
  - BIO（阻塞同步）
    - 连接数少，并发度低，所以Netty已不再支持
  - NIO（非阻塞同步）
    - 不同平台不同实现
  - AIO（非阻塞异步）
    - Linux中相比NIO性能没有明显提升，所以Netty已不再支持
- Reactor模式
  - 核心思想
    - 事件注册
    - 事件监听
    - 事件处理
  - 三种Reactor模式（NioEventLoopGroup）
    - 单线程模式
      - 由一个线程负责处理所有事件
    - 多线程模式
      - 由一个线程池来处理decode/compute/encode三种操作
    - 主从多线程模式
      - 将acceptor连接事件注册到单独的reactor中，其他事件注册到另外的reactor中
- 对TCP粘包和半包的支持
  - 根本原因
    - TCP是流式协议，消息无边界
    - 提醒：UDP消息有边界，无半包粘包问题
  - 解决方式
    - TCP连接改为短连接：按请求次数划分消息边界
    - 封装成帧（Framing）
      - 固定长度：空间浪费
      - 分隔符：需要转义
      - 固定长度标记消息长度：需要提前预知最大长度
  - netty中对Framing操作的支持
    - |方式|解码|编码|
      |---|---|---|
      |固定长度|FixedLengthFrameDecoder||
      |分隔符|DelimiterBasedFrameDecoder||
      |固定长度标记消息长度|LengthFieldBasedFrameDecoder|LengthFieldPrepender|
  