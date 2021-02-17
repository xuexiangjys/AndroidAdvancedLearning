
# XPage页面框架

---

## 没框架的Fragment使用

* 使用FrameLayout作为占位容器

* 使用`FragmentManager`进行管理

```
FragmentManager fragmentManager = getSupportFragmentManager();
FragmentTransaction transaction = fragmentManager.beginTransaction();
transaction.replace(R.id.fl_container, FirstFragment.newInstance());
transaction.commit();
```

---

## Navigation

* 创建导航图

* Fragment占位容器

* 获取NavController进行导航

```
NavHostFragment.findNavController(FirstFragment.this)
        .navigate(R.id.action_FirstFragment_to_SecondFragment);
```

---

## XPage

* 使用`@Page`进行标注。

* 直接调用`openPage`方法打开。

```
openPage(TestFragment.class);
```

---

## 欢迎关注

* 本期内容的源码地址：https://github.com/xuexiangjys/Navigation_XPage

* XPage：https://github.com/xuexiangjys/XPage

* 微信公众号：我的Android开源之旅【openandroidxx】





