# Android 动画框架实现原理

Animation 框架定义了透明度，旋转，缩放和位移几种常见的动画，而且控制的是整个 View，实现原理是每次绘制视图时在 `View.draw(Canvas canvas, ViewGroup parent, long drawingTime)`方法中获取该 View 的 Animation 的 Transformation 值，然后调用 canvas.concat(transformToApply.getMatrix())，通过矩阵运算完成动画帧，如果动画没有完成，继续调用 invalidate() 函数，启动下次绘制来驱动动画，动画过程中的帧之间间隙时间是绘制函数所消耗的时间，可能会导致动画消耗比较多的 CPU 资源，最重要的是，动画改变的只是显示，并不能相应事件。

# Android 各个版本API的区别

只说些主要的:
- 4.4

  - 添加了新的全屏沉浸模式
  - 存储访问框架
  - 透明系统 UI 样式
  - 基于 Chromium 的 WebView 的全新实现
  [官网](https://developer.android.com/about/versions/kitkat.html?hl=zh-cn)
  

- 5.0

  - 把ART模式作为默认的运行模式
  - 引入了Material Design并提供了UI工具包
  - 引入了全新的 Camera API
  - 引入了 Android 扩展包 (AEP)，支持 OpenGL ES 3.1
  - 允许应用利用蓝牙低能耗 (BLE) 执行并发操作的 API，可同时实现扫描（中心模式）和广播（外设模式）
  [官网](https://developer.android.com/about/versions/lollipop.html?hl=zh-cn)

- 6.0

  - 引入了一种新的权限模式 运行时权限检查
  - 取消支持 Apache HTTP 客户端
  - 新的通知构建方法
  - 增加了更强的蓝牙以及 WIFI 保护权限
  [官网](https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html?hl=zh-cn)

- 7.0

  - 系统权限更改，提高了私有文件的安全性
  > 传递软件包网域外的 file:// URI 可能给接收器留下无法访问的路径。因此，尝试传递 file:// URI 会触发 FileUriExposedException。分享私有文件内容的推荐方法是使用 FileProvider。
  - 移除了三项隐式广播

    - CONNECTIVITY_ACTION
    - ACTION_NEW_PICTURE
    - ACTION_NEW_VIDEO

  [官网](https://developer.android.com/about/versions/nougat/android-7.0-changes.html?hl=zh-cn)
- 8.0

  - 后台服务限制，位于后台的应用无法使用 `startService()` 
  - 应用无法使用其清单注册大部分隐式广播
  - 降低了后台应用接收位置更新的频率
  
  [官网](https://developer.android.com/about/versions/oreo/android-8.0-changes.html?hl=zh-cn)

# Requestlayout，onlayout，onDraw，DrawChild区别与联系

- RequestLayout()

  当 view 的大小、位置发生变化时调用 RequestLayout 请求重新布局，将变化后的 View 更新到屏幕上。如果子 View 调用了该方法，那么会从 View 树重新进行一次测量、布局、绘制这三个流程。所以会调用 onMeaue、 onLayout、onDraw，其中 onDraw 的调用可能会也可能不会

- DrawChild()

  DrawChild 会去回调没一个子视图的 draw 方法

# invalidate 和 postInvalidate 的区别及使用

都是用来刷新界面的，不同的是 invalidate 不能在工作线程中使用，只能在主线程中使用， postInvalidate 可以在工作线程中使用。
# Activity-Window-View三者的差别

首先能知道的是 view 的添加删除等是通过 window 来实现的，所以 view 的呈现必须通过 window 来实现。window 通过 windowManager 来管理 view。

window 是一个抽象的概念，它的实现是 phoneWindow。通过 acitivity 的启动过程可以知道 window 的初始化是在 activity 的初始化过程中实现的。

所以 activity 是用来管理 window 的，而 window 则是用来呈现 view，同时 activity 还要用来管理 view 的一些回调。

# 谈谈对 Volley 的理解

Volley 是 Google 官方出的一套小而巧的异步请求库，该框架封装的扩展性很强，支持 HttpClient、HttpUrlConnection，甚至支持 OkHttp。而且 Volley 里面也封装了 ImageLoader ，所以如果你愿意你甚至不需要使用图片加载框架，不过这块功能没有一些专门的图片加载框架强大，对于简单的需求可以使用，对于稍复杂点的需求还是需要用到专门的图片加载框架。

Volley 也有缺陷，比如不支持 post 大数据，所以不适合上传文件。不过 Volley 设计的初衷本身也就是为频繁的、数据量小的网络请求而生！

# 如何优化自定义 View

[优化](http://hukai.me/android-training-course-in-chinese/ui/custom-view/optimize-view.html)

[优化2](http://ibigerbiger.me/2016/10/20/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%28%E4%BA%8C%29-%E8%87%AA%E5%AE%9A%E4%B9%89View%E4%BC%98%E5%8C%96/)

- 不要在 onDraw 中创建对象
- 尽可能的减少 onDraw 的调用次数
- 尽量减少 view 的层级

# 低版本SDK如何实现高版本api？

可以使用 @TargeApi($API_LEVEL) 

或者在运行时判断版本，只有符合版本时才进行调用，同时在低版本中提供相应的处理办法。比如将高版本的部分代码拷贝出来进行改造等。

# 描述一次网络请求的流程

域名解析、TCP的三次握手、建立TCP连接后发起HTTP请求、服务器响应HTTP请求、浏览器解析html代码，同时请求html代码中的资源（如js、css、图片等）、最后浏览器对页面进行渲染并呈现给用户



