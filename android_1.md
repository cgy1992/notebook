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

个人理解 contentProvider 的没有明确的生命周期，只有 oncreate 在被 ContentResolver 第一次调用时回调一次，其存在应该是跟着整个手机开机关机之间的生命进程。如果进程被杀死，这个 contentProvider 也会结束掉。这个时候其他的应用程序使用它时会报错

> Failed to find provider info for ****
> Unknown URI content:****

contentProvider 也可以起到进程间通讯的作用，用于在不同应用程序之间共享数据，比如通讯录等。

## broadcastReceiver

广播也是一种通讯机制，同样支持跨进程通讯。

BroadcastReceiver 是对发送出来的 Broadcast 进行过滤、接受和响应的组件。首先将要发送的消息和用于过滤的信息（Action，Category）装入一个 Intent 对象，然后通过调用 Context.sendBroadcast() 、 sendOrderBroadcast() 方法把 Intent 对象以广播形式发送出去。 广播发送出去后，已注册的 BroadcastReceiver 会检查注册时的 IntentFilter 是否与发送的 Intent 相匹配，若匹配则会调用 BroadcastReceiver 的 onReceiver() 方法。

BroadcastReceiver 的生命周期很短，在执行 onReceiver 才有效，执行结束生命周期就结束了。

建议只在应用内使用的广播用本地广播，更高效和安全。广播分为两种，标准广播和有序广播。

广播的使用

- 静态注册

  需要在清单文件中进行注册

  ```xml
        <receiver android:name=".receiver.NormalReceiver">
            <intent-filter>
                <action android:name="com.demo.kotlintest.NormalReceiver" />
            </intent-filter>
        </receiver>
  ```
- 动态注册

  使用 context.registerReceiver 进行注册

  ```java
  registerReceiver(DynamicReceiver(),IntentFilter("com.demo.kotlintest.NormalReceiver"))
  unregisterReceiver(DynamicReceiver())  
  ```
  registerReceiver 和 unregisterReceiver 一定要成对出现

- 本地广播注册

  使用本地广播的时候必须使用 LocalBroadcastManager 动态注册
  ```java
  LocalBroadcastManager.getInstance(this).registerReceiver(LocalReceiver(), IntentFilter("com.demo.kotlintest.NormalReceiver"))
  LocalBroadcastManager.getInstance(this).unregisterReceiver(LocalReceiver())
  ```
发送广播

- 无序广播

  ```java
            val intent = Intent()
            intent.action = "com.demo.kotlintest.NormalReceiver"
            intent.putExtra("msg", "消灭人类暴政，世界属于三体！！")
            LocalBroadcastManager.getInstance(context!!).sendBroadcast(intent) // 本地广播只限当前应用内使用
            sendBroadcast(intent)  // 属于全局的广播
  ```

- 无序广播

  在未指定优先级的时候动态注册的接收者要比静态注册的优先级高

  ```java
  // 和无序广播一样，第二个参数表示权限
  sendOrderedBroadcast(intent,null) 
  ```

# Activity之间的通信方式

- Intent
- 广播
- ContentProvider
- SharedPreferences
- 文件共享
- 静态变量
- SQLite
等等

# Activity各种情况下的生命周期

典型的声明周期就不说了，正常走完一生

- 切换横竖屏

  不设置 Activity 的 android:configChanges 时，切屏会重新调用各个生命周期默认首先销毁 当前 activity,然后重新加载。

  设置 Activity android:configChanges="orientation|keyboardHidden|screenSize"时，切 屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法。直接设置屏幕方向可以免去这个问题。

- 按 Home 键 再返回

  onPause -> onStop -> onRestart -> onStart -> onResume

- 切换 activity，如果后面的 activity 主题为透明，再返回

  onPause -> onResume

[参考](https://www.jianshu.com/p/e46d449467d5)

# Activity与Fragment之间生命周期比较

![fragment](https://camo.githubusercontent.com/bd18ae36cfba1983a83ca3de4562b452f0591c3b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323234343638312d333638356130383636656230376433612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f333137)

# Activity上有Dialog的时候按Home键时的生命周期 

  onPause -> onStop -> onRestart -> onStart -> onResume

是否有 dialog 不影响 activity 的生命周期

# 两个Activity 之间跳转时必然会执行的是哪几个方法？

第一个 activity 的 onPause，与第二个 activity 的 onCreate onStart onResume

极端情况下，只有 onPause，与第二个onCreate，比如在第二个 activity 的 onCreate 中 finish。

# 前台切换到后台，然后再回到前台，Activity生命周期回调方法。弹出Dialog，生命值周期回调方法 

  onPause -> onStop -> onRestart -> onStart -> onResume

是否有 dialog 不影响 activity 的生命周期

# Activity的四种启动模式对比

- standard 默认模式（标准模式），每次都会新建一个实例。
- slingTop 当需要启动的 activity 位于栈顶的时候不会重复创建，不位于栈顶的时候和 standard 一样。
- slingTask 
  - 如果不指定taskAffinity属性，则会一直在同一个任务栈中，当前任务栈中如果有要启动的 activity，则不会重复创建。会把它之上的activity出栈，并将要启动的 activity 移至栈顶。
  - 如果指定了taskAffinity属性，则会在一个新的任务栈中创建，而且通过此 activity 启动的 activity 也会在这个新的任务栈中。

    但是此时如果使用 startActivityForResult 启动，即使设置了 taskAffinity 属性，也不会创建新的任务栈！！！但是在其他 activity 中再启动这个 activity，会创建新的任务栈，而且此时会存在两个 activity，分别存在于两个任务栈中。

- slingInstance 
  - 以 singleInstance 模式启动的 Activity 具有全局唯一性，即整个系统中只会存在一个这样的实例
  - slingInstance 模式启动的 activity 会独占一个任务栈。
  - 被 singleInstance 模式的 Activity 开启的其他 activity，会在新的任务中启动，但不一定开启新的任务，也可能在已有的一个任务中开启

  实际验证如果以 startActivityForResult 启动一个 slingInstance 模式的 activity,则会和启动它的那个 activity 同一个任务栈，而此时这个 activity 再启动其他的 activity 会使这个 activity 在一个新的任务栈中，并且是被以 slingTask 模式启动。

[参考](https://github.com/maintel/notebook/blob/master/android/activity%E7%9A%84%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F.md)

# Activity状态保存与恢复

- onSaveInstanceState

  在 onSaveInstanceState 中保存信息，
  onSaveInstanceState 会在 onPause 后 onStop 前调用,切换 activity 等都会调用此方法。

- onRestoreInstanceState

  在 onRestoreInstanceState 中恢复。

# fragment各种情况下的生命周期

[参考](https://blog.csdn.net/Jokeeeeee/article/details/46004931)

# 如何实现Fragment的滑动

如果是多 fragment 就是使用 viewpager

# fragment之间传递数据的方式

- bundle进行参数传递，这样在两个Fragment跳转的时候就可以带上参数（仅限于跳转传参）
- 直接拿到 fragment 的实例进行调用
- 通过接口回调
- 通过 EventBus 等第三方框架

# Activity 怎么和 Service 绑定

通过 bindService 方法绑定。绑定成功后会在 ServiceConnection.onServiceConnected 进行回调，并返回一个 Binder 对象，客户端可以使用这个 Binder 对象来与 service 进行通讯。

[绑定原理](https://blog.csdn.net/Luoshengyang/article/details/6745181)

# 怎么在 Activity 中启动自己对应的 Service

可以通过 startService 或者 bindService 来启动相应的 Service。

# Service 和 Activity 怎么进行数据交互

- 通过广播
- 无跨进程时，通过 bindService 得到的 Binder 对象得到 Service 的外部接口进行通讯
- 有跨进程时，通过 bindService 得到的 Binder 对象得到的 Messenger 来进行通讯，或者通过手动实现 AIDL 来进行通讯
- 通过文件共享等

# Service 的开启方式

startService 或 bindService

# 请描述一下 Service 的生命周期

Service 根据不同的启动方式生命周期不太一样

- startService 

![](https://camo.githubusercontent.com/39d4f2f3f2a3eb9ea466b122b443d9d5c7b39fbe/68747470733a2f2f7773312e73696e61696d672e636e2f6c617267652f303036744e633739677931666f77356466776c686b6a333137323062657768372e6a7067)

![](https://camo.githubusercontent.com/d5585e336b851790da62d3b3abe3b0bf1fda26af/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e633739677931666f77356465667364726a33313730306669676f782e6a7067)

- bindService

![](https://camo.githubusercontent.com/bec5a50f49a8b40474eb6f85228e5c00bc19de6c/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e633739677931666f773563366a356c6c6a333137633039777767382e6a7067)

![](https://camo.githubusercontent.com/87089c6ea9aa5d78e8dd043d92c130212b53f0d9/68747470733a2f2f7773312e73696e61696d672e636e2f6c617267652f303036744e633739677931666f77356338377539306a33313636306267676e652e6a7067)

![](https://camo.githubusercontent.com/b49bbc856c9ca929fc4a935e6a1780123cdf5b24/68747470733a2f2f7773312e73696e61696d672e636e2f6c617267652f303036744e633739677931666f7735716c77306f776a333137773035673735352e6a7067)

# 谈谈你对 ContentProvider 的理解

ContentProvider 是一种跨进程通讯的手段，用来很好的解耦不同应用间的数据依赖问题，它可以看做是一个应用为对外提供数据而暴露的抽象接口，既保证了灵活性，也保证了安全性。

# 说说 ContentProvider、ContentResolver、ContentObserver 之间的关系

- ContentProvider 内容提供者，主要来提供以及操作数据
- ContentResolver 用来统一管理当前应用与不同 ContentProvider 之间的操作
- ContentObserver 用来监听目标 ContentProvider 的数据变化

# 请描述一下广播 BroadcastReceiver 的理解

BroadcastReceiver 也可以看作是一个跨进程通讯的手段，用来监听不同的广播以做出相应的操作，或者用来进行组件间的通讯。

# 广播的分类

个人理解可以分成 本地广播和全局广播，同时也有有序广播和无序广播之分。

# 广播使用的方式和场景

使用 context.sendBroadcast 或者其他来发送无序或者有序广播。可以定义广播接收者（BradcastReceiver）来以用来处理广播。

- 在进行跨进程通讯，或者一些耦合性很低的组件之间进行通讯的时候可以使用广播
- 监听系统的广播以优化 App 时，比如监听网络变化，电量变化等

# 本地广播和全局广播有什么差别？

全局广播底层使用 binder 实现，是主要针对应用间，进程间的通信的方式。
本地广播底层使用 handler 来实现，仅限于应用内部进行通信，不能进跨进程的通信。

# 在 manifest 和代码中如何注册和使用 BroadcastReceiver

- 静态注册

  需要在清单文件中进行注册

  ```xml
        <receiver android:name=".receiver.NormalReceiver">
            <intent-filter>
                <action android:name="com.demo.kotlintest.NormalReceiver" />
            </intent-filter>
        </receiver>
  ```
  静态注册随着应用的启动而启动，在应用退出后还能继续工作，onReceive 结束后会自动注销，所以每次接到广播都会生成新的实例。

- 动态注册

  使用 context.registerReceiver 进行注册

  ```java
  registerReceiver(DynamicReceiver(),IntentFilter("com.demo.kotlintest.NormalReceiver"))
  unregisterReceiver(DynamicReceiver())  
  ```
  registerReceiver 和 unregisterReceiver 一定要成对出现，所以应用退出后不能继续工作。

  # BroadcastReceiver，LocalBroadcastReceiver 区别

  BroadcastReceiver 可以使用静态或者动态的方式注册，LocalBroadcastReceiver 只能使用动态注册，发送注册等必须使用 LocalBroadcastManager 的方法。




