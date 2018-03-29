# 四大组件是什么

四大组件：

- activity

  单独的，可以与用户交互的东西，承担着创建 windows 的重任。

  [参考](https://cloud.tencent.com/developer/article/1028813)
- service

  为应用程序运行一些需要长时间运行的操作，不直接参与与用户的交互
  为其他应用提供一些功能（提供能够跨进程调用的功能）

  - service不是一个单独的进程。除非特别说明，否则service不会运行在他自己的进程中，而是运行在应用进程中。
  - service不是一个线程。service也不是脱离主线程而运行的线程

  [参考](https://cloud.tencent.com/developer/article/1028811)
- contentProvider

  进程间 进行数据交互 & 共享，即跨进程通信，底层是采用 Android中的Binder机制。

  [参考](https://blog.csdn.net/carson_ho/article/details/76101093)

- broadcastReceiver

  广播接收者，是组件间传播信息的一种方式，同样也可以起到进程间通讯的作用。

  [参考](https://www.jianshu.com/p/f348f6d7fe59)

# 四大组件的生命周期和简单用法

## activity

![activity 的生命周期](https://ask.qcloudimg.com/http-save/yehe-1216241/zy4scysvwz.png?imageView2//0/w/1620)

与用户交互的界面都要依赖于 activity，使用在清单文件中注册，然后继承一个 activity ，setContentView() 就可以了。

## service

service 根据启动方式的不同生命周期也不太一样。

通过 strart 的方式启动的 

> onCreate()--->onStartCommand()（onStart()方法已过时） ---> onDestory()

多次启动不会重复执行 onCreate ，但是会重复执行 onStartCommand

通过 bind 启动的

> onCreate() --->onBind()--->onunbind()--->onDestory()

多次绑定一般不会重复执行 onBind 方法

![service 的生命周期](https://ask.qcloudimg.com/http-save/yehe-1216241/6qnnsbpx93.png?imageView2//0/w/1620)

同时，根据不同的启动方法以及 service 的类型不同，通讯机制也不一样。

- 如果是 start 方式启动的 service 使用接口或者广播接收者等来进行通讯

- 如果是 bind 方式启动的本地 service 可以使用 ServiceConnection.onServiceConnected 返回的 Ibinder 接口进行通讯。

- 如果是 bind 方式启动的远程 service 同样要是 ServiceConnection.onServiceConnected 返回的 Ibinder 接口，但是此时 Ibinder 是一个 binderProxy 对象，建议使用 messenger 进行通讯，这个涉及到 IPC 通讯机制。

## contentProvider

个人理解 contentProvider 的没有明确的生命周期，只有 oncreate 在被 ContentResolver 第一次调用时回调一次，其存在应该是跟着整个手机开机关机之间的生命进程。
