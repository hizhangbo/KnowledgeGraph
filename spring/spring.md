- Spring BeanFactory 与 FactoryBean 的区别
  - BeanFactory 和 FactoryBean 都是Spring Beans模块下的接口
  - BeanFactory是spring简单工厂模式的接口类，spring IOC特性核心类，提供从工厂类中获取bean的各种方法，是所有bean的容器。
  - FactoryBean仍然是一个bean，但不同于普通bean，它的实现类最终也需要注册到BeanFactory中。
  - 实现了FactoryBean接口的类，在注册到spring BeanFactory后，并不像其它类注册后暴露的是自己，它暴露的是FactoryBean中getObject方法的返回值。
  - 总结
    - BeanFactory是IOC容器，FactoryBean是简化复杂对象的创建工厂。
- IOC
  - Bean生命周期
    - Singleton：容器启动时创建，容器关闭时销毁
    - Prototype：仅在使用时创建，使用完毕后等待GC销毁
- AOP
  - JDK动态代理（java.lang.reflect.Proxy）
    - 反射生成动态代理类
  - cglib
    - 编译时字节码动态织入

- Transaction
  - 传播类型（7种）
    - Required 没有事务创建事务，有事务则合并事务
    - Supports 没有事务以非事务方式执行，有事务则在该事务中
  - 隔离类型
    - Serializable 串行
    - Repeatable Read 可重复读
    - Read Committed 提交后读
    - Read Uncommitted 不提交读
    

## WebFlux
 - 相关前提
   - Reactor 同步非阻塞模式
   - Proactor 异步非阻塞模式
   - Observer 观察者模式
   - Iterator 迭代器模式
 - Reactive
   - 定义
     - 声明式的编程范式，关注于数据流（data streams）和传播变化（propagation of change）
   - 实现
     - RxJava: Reactive Extensions
     - Reactor: Spring WebFlux Reactive 
     - Flow API: JAVA 9 Flow API
   - 特点
     - 响应式（Responsive）
     - 适应性强（Resilient）
     - 弹性可伸缩（Elastic）
     - 消息驱动（Message Driven）
   - 涉及技术
     - 数据流：Java 8 `Stream`
     - 传播变化： Java `Observable/Observer`
     - 事件： Java `EventObject/EventListener`
   - 数据结构
     - 流式 `stream`
     - 序列 `Sequences`
     - 事件 `Events`
   - 设计模式
     - 扩展模式：观察者 `Observer` -> 推模式：被动获取事件
     - 对立模式：迭代器 `Iterator` -> 拉模式：主动获取事件
     - 混合模式：反应堆 `Reactor`,`Proactor`
     
     
     