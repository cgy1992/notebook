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

