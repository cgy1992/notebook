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