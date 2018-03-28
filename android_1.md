# 四大组件是什么

四大组件：

- activity

  单独的，可以与用户交互的东西，承担着创建 windows 的重任。

  [参考](https://cloud.tencent.com/developer/article/1028813)
- service

  为应用程序运行一些需要长时间运行的操作，不直接参与与用户的交互
  为其他应用提供一些功能（提供能够跨进程调用的功能）

  [参考](https://cloud.tencent.com/developer/article/1028811)
- contentProvider

  进程间 进行数据交互 & 共享，即跨进程通信，底层是采用 Android中的Binder机制。

  [参考](https://blog.csdn.net/carson_ho/article/details/76101093)

- broadcastReceiver

  
