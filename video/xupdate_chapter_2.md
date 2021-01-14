# XUpdate

# 框架基础使用详解

---

## 使用简化库API

* 配置`jitpack`仓库

* 增加`XUpdateAPI`依赖

---

## 简单使用

* 使用`EasyUpdate`
    * EasyUpdate.create: 构建版本更新检查管理者
    * EasyUpdate.checkUpdate: 直接版本更新

* 使用`AriaDownloader`断点续传

---

## 自定义初始化配置

* 实现`IUpdateConfigProvider`接口

---

## 注意事项

* 服务器Json格式

```json
{
  "Code": 0, // 0代表请求成功，非0代表失败
  "Msg": "", // 请求出错的信息
  "UpdateStatus": 1, // 0代表不更新，1代表有版本更新，不需要强制升级，2代表有版本更新，需要强制升级
  "VersionCode": 3,
  "VersionName": "1.0.2",
  "ModifyContent": "1、优化api接口。\r\n2、添加使用demo演示。\r\n3、新增自定义更新服务API接口。\r\n4、优化更新提示界面。",
  "DownloadUrl": "https://raw.githubusercontent.com/xuexiangjys/XUpdate/master/apk/xupdate_demo_1.0.2.apk",
  "ApkSize": 2048,
  "ApkMd5": "..."  // md5值没有的话，就无法保证apk是否完整，每次都会重新下载。
}
```

---

* 混淆配置

    * XUpdate
    
    * AriaDownloader

---

## 欢迎关注我

![](https://img.rruu.net/image/5f871cffe209c)

* 微信公众号：我的Android开源之旅
* Github：https://github.com/xuexiangjys
* 喜欢的话，一定记得三连支持一下哦！

