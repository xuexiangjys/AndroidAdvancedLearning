# 广播的注册、发送和接收过程

> 广播的注册、发送和接收都与AMS有着密不可分的关系。

## 广播的注册

> 广播的注册可分为静态注册和动态注册两种，静态注册在应用安装时由`PackageManagerService`来完成注册过程，下面我主要来分析动态广播注册。

### ContextImpl请求AMS注册广播

![](../img/register_receiver.png)

* 当我们需要动态注册广播时，需要调用Context的`registerReceiver`方法，然后在[ContextWrapper](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/content/ContextWrapper.java#621)的`registerReceiver`中调用[ContextImpl](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ContextImpl.java#105)的`registerReceiver`方法，最终会调用其[registerReceiverInternal](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ContextImpl.java#1467)方法。

* 在ContextImpl的`registerReceiverInternal`方法中，首先是和服务绑定类似的，通过`LoadedApk`类型的mPackageInfo对象的`getReceiverDispatcher`方法来获取`IIntentReceiver`类型的rd对象，用于广播的跨进程通信。然后调用IActivityManager的`registerReceiver`方法，最终调用AMS的`registerReceiver`方法，并将`IIntentReceiver`类型的rd对象传入。

* 在AMS的[registerReceiver](http://androidxref.com/9.0.0_r3/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java#20801)方法中，首先是调用`getRecordForAppLocked`方法获取调用注册广播的应用程序进程信息，然后根据进程信息获取对应在AMS中存储的所有粘性广播的intent，然后和传入的参数filter的粘性广播进行对比，找到所有匹配的intent存入到allSticky列表中，最终加入到广播队列中执行。

* 除此之外，在AMS的`registerReceiver`中还调用了HashMap类型，存放了所有应用进程的广播接收者列表mRegisteredReceivers，通过传入之前的`IIntentReceiver`对象获取到对应的广播接收者列表ReceiverList，并将其传入创建BroadcastFilter，用以描述注册的广播接收者。最后将BroadcastFilter添加到IntentResolver类型的mReceiverResolver中，这样当AMS接收到广播时，就可以从mReceiverResolver中直接找到对应的广播接收者，从而达到注册广播的目的。


## 广播的发送




## 广播的接收