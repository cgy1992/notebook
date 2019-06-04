基于 leakCanary 1.6.1 版本。

程序入口 LeakCanary.install(this);

```java
  public static RefWatcher install(Application application) {
    return refWatcher(application).listenerServiceClass(DisplayLeakService.class)
        .excludedRefs(AndroidExcludedRefs.createAppDefaults().build())
        .buildAndInstall();
  }
```


