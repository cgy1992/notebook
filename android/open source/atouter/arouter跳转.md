
# 跳转
## 简单跳转
跳转分为两步

- 首先在目标 activity 上添加注解

```kotlin
@Route(path = "/main/mainActivityTest")
class MainTestActivity : AppCompatActivity() {
}
```

这里的路径一定要写两级，为什么下面会说

- 然后就可以通过指定路径进行跳转

跳转的方式
```kotlin 
// 这种方法已经过时
ARouter.getInstance().build("/main/mainActivityTest","main").navigation()
// 路径应该写两级
ARouter.getInstance().build("/main/mainActivityTest").navigation()
```

可以看到有两种方式，
- 第一种方式 两个参数分别指定了 路径 和 组名

- 第二种方式会自动从路径中解析组名，看一下源码

```java
    /**
     * Build postcard by path and default group
     */
    protected Postcard build(String path) {
           ...
            return build(path, extractGroup(path));
    }

    private String extractGroup(String path) {
      ...
        try {
            String defaultGroup = path.substring(1, path.indexOf("/", 1));
            if (TextUtils.isEmpty(defaultGroup)) {
                throw new HandlerException(Consts.TAG + "Extract the default group failed! There's nothing between 2 '/'!");
            } else {
                return defaultGroup;
            }
        } catch (Exception e) {
            logger.warning(Consts.TAG, "Failed to extract default group! " + e.getMessage());
            return null;
        }
    }
```

可以知道，它默认路径中第一级为组名，所以建议使用第二种方式来进行跳转，可以少些一些代码，同时也更方便管理。

## 携带参数

直接使用 withXXX() 来添加参数，和 使用 intent 一样，支持多种类型。

![](http://blogqn.maintel.cn/TIM截图20181122171334.png?e=3119678170&token=cs2nCfx72Y7hW0_NpFYzb3Jab90IJWraRtphMd-q:w02Cgeb1qqZwN3-ZhthSp4R16O4=)

```kotlin
            ARouter.getInstance().build(ACTIVITY_MAIN_TEST)
                    .withString("test", "传递一个字符串过来")
                    .navigation()
```

接收的时候和使用 intent 并无区别。

## 使用 uri 进行跳转

arouter 可以通过监听 Schame 的方式进行跳转

```kotlin
            val uri = Uri.parse("arouter://m.maintel.cn/test/schameTest")
            ARouter.getInstance().build(uri).navigation()
```

目标 activity 可以不用注册 schame 但是一样要指定 Route 注解

```kotlin
@Route(path = "/test/schameTest")
class SchameTestActivity{}
```

并且 路径要保持一致才能跳转。

个人感觉这个主要作用是用在 通过schame 注册一个中间页，然后通过路径转发，这样就不用再很多页面注册 schame 了。

### 使用 Uri 跳转时自动注入参数

Arouter 可以通过解析 url 中的参数来对目标 activity 进行自动注入，如下面这样：

跳转

```kotlin
val uri = Uri.parse("arouter://m.maintel.cn/test/schameTest?name=maintel&age=100")
ARouter.getInstance().build(uri).navigation()
```


```kotlin
@Route(path = "/test/schameTest")
class SchameTestActivity : AppCompatActivity() {

    // 可以通过 name 字段指定具体需要解析的字段
    @Autowired(name = "name")
    @JvmField
    public var school = ""
    @Autowired
    @JvmField
    public var age = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ARouter.getInstance().inject(this)
        val textview = TextView(this)
        textview.text = "$school::$age"
        setContentView(textview)
    }
}
```

但是注意必须是 public 修饰的属性，这个在 kotlin 中要注意，虽然 kotlin 中属性默认是 public 的，但是 kotlin 的 public 和 java 上的意义不同，在编译器，通过 kapt 编译器来扫描注解的时候 kotlin 代码会转化成对应的 java 代码，这个时候它还是 private 的，解决办法也很简单，在需要注入的属性上加入 @JvmField 注解。

关于 kapt 转换后的代码可以在 app/build/temp/kapt3/stubs/debug/包名下找到。

```java
@com.alibaba.android.arouter.facade.annotation.Route(path = "/test/schameTest")
@kotlin.Metadata(...)
public final class SchameTestActivity extends androidx.appcompat.app.AppCompatActivity {
    @org.jetbrains.annotations.NotNull()
    @com.alibaba.android.arouter.facade.annotation.Autowired(name = "\u6e05\u534e")
    private java.lang.String school;
    ...
}
```

在 Url 中不支持传递 Parcelable 类型的数据，但是支持解析自定义对象，需要用json 字符串来传递。

这个时候需要添加一个用来序列化器，实现 SerializationService 并使用 route 注解标注。

```kotlin
@Route(path = "/jsonService/json")
class JsonServiceImpl : SerializationService {
    override fun <T : Any?> json2Object(input: String?, clazz: Class<T>?): T {
        return Gson().fromJson(input, clazz)
    }

    override fun init(context: Context?) {

    }

    override fun object2Json(instance: Any?): String {
        return Gson().toJson(instance)
    }

    override fun <T : Any?> parseObject(input: String?, clazz: Type?): T {
        return Gson().fromJson(input, clazz)
    }
}
```

值得注意的是，这个序列化器只能创建一个，创建多个是没用的，意思就是没办法指定具体使用什么序列化器来执行序列化，即使他们的组名不一样，具体可以参考它生成的 ARouter$$Providers$$app 类：

```java
public class ARouter$$Providers$$app implements IProviderGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> providers) {
    providers.put("com.alibaba.android.arouter.facade.service.SerializationService", RouteMeta.build(RouteType.PROVIDER, JsonServiceImplTest.class, "/test/json", "test", null, -1, -2147483648));
    providers.put("com.alibaba.android.arouter.facade.service.SerializationService", RouteMeta.build(RouteType.PROVIDER, JsonServiceImpl.class, "/main/json", "main", null, -1, -2147483648));
  }
}
```

可以看到，service 会存在一个map 中，但是 key 是相同的，后面的会覆盖掉前面的类。


然后使用 json 的方式传递参数即可，例如下面这样子。

```kotlin
val uri = Uri.parse("arouter://m.maintel.cn/test/schameTest?name=maintel&age=100&student={name:\"泰迪\",age:20}")
```
