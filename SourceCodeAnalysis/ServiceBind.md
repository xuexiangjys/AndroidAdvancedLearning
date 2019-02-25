# Service的绑定过程

> 除了使用Context的startService来启动Service外，我们也可以通过Context的bindService来绑定Service。绑定Service的过程要比启动Service的过程复杂一些。不清楚[Service启动](./ServiceStartup.md)的可以点击了解一下。

## 启动大纲

1. ContextImpl请求AMS绑定Service.

2. AMS请求ActivityThread处理Service绑定.

3. AMS进行Service的绑定.

---

### ContextImpl请求AMS绑定Service

![](../img/servicebind.png)

* 当我们需要绑定一个Service时，我们会使用`context.bindService`。而Context只是一个抽象的类，它的实现是在[ContextWrapper](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/content/ContextWrapper.java#696)中。

* 在ContextWrapper的`bindService`方法中，又会调用其内部的Context类型的mBase变量，而该变量的创建详见[ActivityThread](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ActivityThread.java#2990)的`createBaseContextForActivity`方法，它的实现类是`ContextImpl`。

* 在[ContextImpl](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ContextImpl.java#1609)的`bindService`方法中，又会调用其自身的`bindServiceCommon`方法。

* 在[ContextImpl](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ContextImpl.java#1651)的`bindServiceCommon`方法中，首先调用[LoadedApk](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/LoadedApk.java#1492)的`getServiceDispatcher`方法获取ServiceConnection的封装类`IServiceConnection`(用于跨进程通信)，然后使用`ActivityManager`获取AMS的代理`IActivityManager`，调用其`bindService`方法并将`IServiceConnection`对象传入。

### AMS请求ActivityThread处理Service绑定