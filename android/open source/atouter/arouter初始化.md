
> 本篇主要介绍 ARouter 的初始化过程。

# 初始化

一般建议在应用打开时的初始化过程即在 application 中进行初始化工作。同时如果是在开发的情况下最好打开 log 和 debug 开关。

```kotlin
        if (BuildConfig.DEBUG) {
            ARouter.openLog()
            ARouter.openDebug()
        }
        ARouter.init(this)
```

ARouter.init(this) 是一个单例。它把初始化以及初始化后的工作都交给了 `_ARouter` 来做。

```java
    // class ARouter
    public static void init(Application application) {
        if (!hasInit) {
            logger = _ARouter.logger;
            hasInit = _ARouter.init(application);
            if (hasInit) {
                _ARouter.afterInit();
            }
        }
    }
    // class _ARouter
    protected static synchronized boolean init(Application application) {
        mContext = application;
        LogisticsCenter.init(mContext, executor);
        hasInit = true;
        mHandler = new Handler(Looper.getMainLooper());
        return true;
    }
```

在 `_ARouter.init` 把初始化工作交给了 `LogisticsCenter.init` 来做，同时自己初始化了一个 handler。

# LogisticsCenter.init

在 ARouter 中 `LogisticsCenter` 是一个很关键的类，几乎所有的工作最终都是由它来做。

其中 `init` 方法是扫描所有路由的映射关系，并将它们保存到内存中来。下面是它的部分关键代码：

```java
    public synchronized static void init(Context context, ThreadPoolExecutor tpe) throws HandlerException {
        try {
            ...
            if (registerByPlugin) { // 一般这里就是 false ，除非是使用 ARouter 提供的插件自动设置路由的。
                logger.info(TAG, "Load router map by arouter-auto-register plugin.");
            } else {
                Set<String> routerMap;

                // 如果是 debug 或者升级了新版本，则扫描路由
                // 这也是为什么我们要在开发状态下打开 debug 开关的原因
                if (ARouter.debuggable() || PackageUtils.isNewVersion(context)) {
                    // 扫描 dex 包，找出所有路由
                    routerMap = ClassUtils.getFileNameByPackageName(mContext, ROUTE_ROOT_PAKCAGE);
                    if (!routerMap.isEmpty()) {
                        // 缓存路由
                        context.getSharedPreferences(AROUTER_SP_CACHE_KEY, Context.MODE_PRIVATE).edit().putStringSet(AROUTER_SP_KEY_MAP, routerMap).apply();
                    }
                    // 保存版本
                    PackageUtils.updateVersion(context);  
                } else {
                    // 从缓存中读取
                    routerMap = new HashSet<>(context.getSharedPreferences(AROUTER_SP_CACHE_KEY, Context.MODE_PRIVATE).getStringSet(AROUTER_SP_KEY_MAP, new HashSet<String>()));
                }

                
                for (String className : routerMap) {
                    if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_ROOT)) {
                        // This one of root elements, load root.
                        ((IRouteRoot) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.groupsIndex);
                    } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_INTERCEPTORS)) {
                        // Load interceptorMeta
                        ((IInterceptorGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.interceptorsIndex);
                    } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_PROVIDERS)) {
                        // Load providerIndex
                        ((IProviderGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.providersIndex);
                    }
                }
            }

            logger.info(TAG, "Load root element finished, cost " + (System.currentTimeMillis() - startInit) + " ms.");

            if (Warehouse.groupsIndex.size() == 0) {
                logger.error(TAG, "No mapping files were found, check your configuration please!");
            }

            if (ARouter.debuggable()) {
                logger.debug(TAG, String.format(Locale.getDefault(), "LogisticsCenter has already been loaded, GroupIndex[%d], InterceptorIndex[%d], ProviderIndex[%d]", Warehouse.groupsIndex.size(), Warehouse.interceptorsIndex.size(), Warehouse.providersIndex.size()));
            }
        } catch (Exception e) {
            throw new HandlerException(TAG + "ARouter init logistics center exception! [" + e.getMessage() + "]");
        }
    }
```


上面代码可以看到主要有两个关键步骤：

- 扫描所有的路由
- 将路由关系保存到内存中来

# 扫描路由表

`ClassUtils.getFileNameByPackageName(mContext, ROUTE_ROOT_PAKCAGE);`

```java
//com.alibaba.android.arouter.routes ---->packageName
public static Set<String> getFileNameByPackageName(Context context, final String packageName) throws PackageManager.NameNotFoundException, IOException, InterruptedException {
        final Set<String> classNames = new HashSet<>();

        List<String> paths = getSourcePaths(context);
        final CountDownLatch parserCtl = new CountDownLatch(paths.size());

        for (final String path : paths) {
            DefaultPoolExecutor.getInstance().execute(new Runnable() {
                @Override
                public void run() {
                    DexFile dexfile = null;

                    try {
                        if (path.endsWith(EXTRACTED_SUFFIX)) {
                            //NOT use new DexFile(path), because it will throw "permission error in /data/dalvik-cache"
                            dexfile = DexFile.loadDex(path, path + ".tmp", 0);
                        } else {
                            dexfile = new DexFile(path);
                        }

                        Enumeration<String> dexEntries = dexfile.entries();
                        while (dexEntries.hasMoreElements()) {
                            String className = dexEntries.nextElement();
                            if (className.startsWith(packageName)) {
                                classNames.add(className);
                            }
                        }
                    } catch (Throwable ignore) {
                        Log.e("ARouter", "Scan map file in dex files made error.", ignore);
                    } finally {
                        if (null != dexfile) {
                            try {
                                dexfile.close();
                            } catch (Throwable ignore) {
                            }
                        }

                        parserCtl.countDown();
                    }
                }
            });
        }

        parserCtl.await();

        Log.d(Consts.TAG, "Filter " + classNames.size() + " classes by packageName <" + packageName + ">");
        return classNames;
    }
```


根据以上代码可以知道，它会扫描 dex 包，并从dex 包中扫描出符合 arouter 包名的类，添加到集合当中并返回。

再然后通过 反射的方法创建出这些类，并调用它的 loadInto 方法，把路由映射表保存到内存当中。
```java
        for (String className : routerMap) {
              if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_ROOT)) {
             // 通过反射将映射表组保存到内存中
           ((IRouteRoot) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.groupsIndex);
          } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_INTERCEPTORS)) {
             // 把拦截器加载到内存中
             ((IInterceptorGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.interceptorsIndex);
            } else if (className.startsWith(ROUTE_ROOT_PAKCAGE + DOT + SDK_NAME + SEPARATOR + SUFFIX_PROVIDERS)) {
            // 把服务类加载到内存中
             ((IProviderGroup) (Class.forName(className).getConstructor().newInstance())).loadInto(Warehouse.providersIndex);
             }
     }
```
根据上面的代码可以知道，在初始化的时候都分别加载了这些类

- 保存各个路由组关系的类
- 拦截器被加载到内存中
- 保存各个服务的路由关系的类

其实是保存在了 Warehouse 中。这个类如下：

都是一些全局的 map。

```java
class Warehouse {
    // Cache route and metas
    static Map<String, Class<? extends IRouteGroup>> groupsIndex = new HashMap<>();
    static Map<String, RouteMeta> routes = new HashMap<>();

    // Cache provider
    static Map<Class, IProvider> providers = new HashMap<>();
    static Map<String, RouteMeta> providersIndex = new HashMap<>();

    // Cache interceptor
    static Map<Integer, Class<? extends IInterceptor>> interceptorsIndex = new UniqueKeyTreeMap<>("More than one interceptors use same priority [%s]");
    static List<IInterceptor> interceptors = new ArrayList<>();

    static void clear() {
        routes.clear();
        groupsIndex.clear();
        providers.clear();
        providersIndex.clear();
        interceptors.clear();
        interceptorsIndex.clear();
    }
}
```

再根据最前面的 `ARouter$$Root$$app` 类的 loadInto 方法

```java
  @Override
  public void loadInto(Map<String, Class<? extends IRouteGroup>> routes) {
    routes.put("main", ARouter$$Group$$main.class);
  }
```

它确实把路由映射表按照组的方式保存到了内存当中。需要注意的是，这里只是把某一组的路由映射关系保存到了内存中，因为`ARouter$$Group$$main`中才实际保存了一组 activity 的路由映射关系。而他们的映射关系是在第一个路径被访问的时候才加载到内存中的，这个就是 ARouter 的按需加载机制了。


在以上初始化完成以后

```java
if (hasInit) {
                _ARouter.afterInit();
            }

    static void afterInit() {
        // Trigger interceptor init, use byName.
        interceptorService = (InterceptorService) ARouter.getInstance().build("/arouter/service/interceptor").navigation();
    }
```

初始化了拦截器`com.alibaba.android.arouter.core.InterceptorServiceImpl`



对于正常配置后的经过编译会在 app/build/generated/source/kapt/debug/包名 下生成至少三个类，分别为：

`ARouter$$Group$$组名`

存储某一组的路由映射

`ARouter$$Providers$$app`
`ARouter$$Root$$app`

用于初始化，将路由按组存储到一个 map 中。

上面这一些类是在编译过程中生成的。


