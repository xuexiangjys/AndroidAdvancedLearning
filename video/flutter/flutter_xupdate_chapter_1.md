
# flutter如何实现

# 版本更新功能？

---

## 常见思路

* 1.使用 `package_info` 插件获取当前应用的版本信息。
* 2.使用 `dio` 插件进行网络请求获取最新版本信息。
* 3.使用 `flutter_downloader` 插件下载最新APK。
* 4.使用 `install_plugin` 插件安装APK。

---

## 偷懒的做法

毕竟版本更新是一个相对稳定且独立的功能，每次就为一个小小的版本更新都需要写这么多冗余的代码实在是不够优雅！

那么，有没有一个可以只通过一行代码就可以实现版本更新的插件呢？

---

## 答案就是

### flutter_xupdate插件！

---

## 项目介绍

> flutter_xupdate是一个可以一键实现Android内版本更新的flutter插件。内部集成的是XUpdate。

项目地址: https://github.com/xuexiangjys/flutter_xupdate

---

## 如何集成

1.在你的flutter项目中的`pubspec.yaml`文件中添加`flutter_xupdate`依赖.

```
dependencies:
  flutter_xupdate: ^1.0.2
```

---

2.修改Android项目的主题为`AppCompat`主题，文件路径: `android/app/src/main/res/values/styles.xml`, 例如:

```
<resources>
    <style name="LaunchTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowBackground">@drawable/launch_background</item>
    </style>
</resources>
```

---

## 初始化

* 调用`FlutterXUpdate.init`方法进行初始化.

* 调用`FlutterXUpdate.setErrorHandler`方法设置错误监听.


---

## 如何使用

* 默认版本更新

* 支持后台版本更新

* 自定义版本样式


---

## 自定义Json解析

* 1.定义一个自定义的版本更新解析器

* 2.调用`checkUpdate`方法,并设置`isCustomParse`参数为true.

---

## 欢迎关注我

![](https://ss.im5i.com/2021/06/14/6tqAU.png)

* 微信公众号：我的Android开源之旅
* Github：https://github.com/xuexiangjys
* 喜欢的话，一定记得三连支持一下哦！
