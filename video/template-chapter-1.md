
# 如何使用模版工程

---

## 克隆项目

```
git clone https://github.com/xuexiangjys/TemplateAppProject.git
```

---

## 项目修改

* 修改项目名（文件夹名），并删除目录下的.git文件夹（隐藏文件）

* 修改包名

* 修改applicationId

* 修改app_name

* 修改启动页样式

---

## 打包

* 修改工程根目录的gradle.properties中的isNeedPackage=true。

* 添加并配置keystore，在versions.gradle中修改app_release相关参数。

* 如果考虑使用友盟统计的话，在local.properties中设置应用的友盟ID:APP_ID_UMENG。

* 使用./gradlew clean assembleReleaseChannels进行多渠道打包。


---

## 欢迎关注

* 模版地址：https://github.com/xuexiangjys/TemplateAppProject

* Github：https://github.com/xuexiangjys

* 微信公众号：我的Android开源之旅【openandroidxx】





