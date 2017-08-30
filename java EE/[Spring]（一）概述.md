# Spring

Spring Framework 是一个提供完善的基础设施用来支持来开发 Java 应用程序的 Java 平台。Spring 负责基础设施功能，使程序员可以专注于应用的开发。

简单的来说 Spring 是一个轻量级的 Java 开发框架。通过启用基于 POJO 编程模型来促进良好的编程实践。

使用 Spring 的好处：

- 使 Java 方法可以执行数据库事务而不用去处理事务 API。
- 使本地 Java 方法可以执行远程过程而不用去处理远程 API。
- 使本地 Java 方法可以拥有管理操作而不用去处理 JMX API。
- 使本地 Java 方法可以执行消息处理而不用去处理 JMS API。

# Spring 核心功能

Spring 的核心功能有两个

- Spring容器作为超级大工厂，负责创建、管理所有的Java对象，这些Java对象被称为Bean。
- Spring容器管理容器中Bean之间的依赖关系，Spring使用一种被称为"依赖注入"的方式来管理Bean之间的依赖关系。





spring 在创建指运行开始 通过 ClassPathXmlApplicationContext 就把配置文件中的类全部 new 出来了

并存放在 DefaultSingletonBeanRegistry.singletonObjects 中  singletonObjects 是一个map

![](http://blogqn.maintel.cn/spring 分析 bean加载机制.png?e=3080883743&token=kDSqSAyKGaf8JcHprWP7S4W3hGuz8kDIEhzAufWH:NegbLeefFbemOSGBy7fD8njQFlE=)