<!-- TOC -->

- [implementation() not mathod](#implementation-not-mathod)
- [output.outputFile 报错](#outputoutputfile-%E6%8A%A5%E9%94%99)
- [Could not get unknown property 'apkVariantData' for object of type](#could-not-get-unknown-property-apkvariantdata-for-object-of-type)
- [遇到 Error:Execution failed for task ':xxxxxxx'. 的解决办法](#%E9%81%87%E5%88%B0-errorexecution-failed-for-task-xxxxxxx-%E7%9A%84%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95)

<!-- /TOC -->

# implementation() not mathod 

这个是因为 gradle 版本过低的原因，升级到3.0.0以上就好了

# output.outputFile 报错

在 gradle 3.0 以下版本是没有问题的，但是在gradle 3.0 以上就有问题了。

解决办法，在3.0以上使用 outputs.first().outputFile 来替代。

# Could not get unknown property 'apkVariantData' for object of type 

完整错误

> Error:Could not get unknown property 'apkVariantData' for object of type com.android.build.gradle.internal.api.ApplicationVariantImpl.

这个是因为在2.x中的 getApkVariantData() 函数在3.x中被修改成了 getVariantData() ，所以 ApplicationVariantImpl 类的 apkVariantData 属性就不存在了。

解决办法：降低 gradle 的版本

> 遇到问题的地方是 tinker 中，使用版本的1.7.5，升级到1.9.2版本后解决此问题

# 遇到 Error:Execution failed for task ':xxxxxxx'. 的解决办法

打包或者 build 项目的时候有时候会报这种错误，某个任务执行失败。比如：

    Error:Execution failed for task ':test:processDebugManifest'.
    > Manifest merger failed with multiple errors, see logs

但是又找不到 log 在哪，可以在 Terminal 执行如下命令来获取详细的错误信息：

    gradlew processDebugManifest --stacktrace

即出现什么任务错误，就单独执行什么任务。
