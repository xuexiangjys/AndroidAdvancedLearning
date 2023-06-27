# DataBinding系列之基础使用

## 1.前言

> DataBinding, 又名数据绑定，是Android开发中非常重要的基础技术，它可以将UI组件和数据模型连接起来，使得在数据模型发生变化时，UI组件自动更新，从而节省了大量的代码和时间。

DataBinding的原理是通过编写XML布局文件，在其中使用特定的标签和语法，将UI组件和数据模型连接起来。当布局文件被加载时，DataBinding会自动生成绑定代码，从而将UI组件和数据模型关联起来。 

通过学习DataBinding基础知识，可以让你的代码速度翻倍，提高开发效率和代码质量。因此，如果你希望在Android开发中获得更高的成功率和更快的发展速度，那么请务必学习DataBinding技术，掌握其基础知识，让自己成为一名高效率的Android开发者！

那么话不多说，让我们直接直奔主题。接下来我将从实用性的角度，来逐一讲解DataBinding的基础使用，文章末尾会给出示例代码的链接地址，希望能给你带来启发。

## 2.准备工作

### 2.1 启用

1.DataBinding启用

```groovy
android {
    dataBinding {
        enabled = true
    }
}
```

2.ViewBinding启用

```groovy
android {
    buildFeatures {
        viewBinding true
    } 
}
```

### 2.2 快捷方式

在你的布局中找到最外层的布局，将光标放在如图位置。

* Windows 请按快捷键 `Alt + 回车`
* Mac 请按快捷键 `option + 回车`

![](https://s1.ax1x.com/2023/04/22/p9Z9cgP.png)

![](https://s1.ax1x.com/2023/04/22/p9Z963t.png)

## 3.基础使用

### 3.1 点击事件绑定

1.无参响应函数：

```kotlin
fun onIncrement() {
    // 方法体
}
```

```
android:onClick="@{() -> state.onIncrement()}"
```

2.接口变量响应函数

注意，这里变量的类型应该是`View.OnClickListener`接口。

```kotlin
val onClickDecrement = View.OnClickListener {
    // 方法体
}
```

```
android:onClick="@{state.onClickDecrement}"
```

3.有参响应函数

```kotlin
fun onReset(view: View) {
    // 方法体
}
```

```
// 第一种写法
android:onClick="@{(view) -> state.onReset(view)}" 

// 第二种写法
android:onClick="@{state::onReset}"
```

### 3.2 @BindingAdapter自定义属性

> 所有注解的功能都是基于XML属性值为DataBinding表达式才生效(即@{})

使用@BindingAdapter进行控件自定义属性绑定的时候，一定要使用 `"@{}"` 进行赋值，这一点非常重要！！！

1. 顶级函数实现

```kotlin
// Kotlin拓展函数式写法, 推荐使用
@BindingAdapter("customTitle")
fun TextView.setCustomTitle(title: String) {
    text = "标题1: $title"
}

// 第一个参数必须是view的子类
@BindingAdapter("customTitle1")
fun setCustomTitle1(view: TextView, title: String) {
    view.text = "标题2: $title"
}

// 多个参数进行绑定，requireAll=true，代表两个参数都设置了才生效，默认是true.
// 如果requireAll为false, 你没有填写的属性值将为null. 所以需要做非空判断.
@BindingAdapter(value = ["customTitle", "customSize"], requireAll = true)
fun TextView.setTextContent(title: String, size: Int) {
    text = "标题3: $title"
    textSize = size.toFloat()
}
```

**【特别注意事项⚠️】**

很多时候，很多新手在写DataBinding的时候，经常会漏掉`"@{}"`，尤其是用数字和Boolean类型的值时。就比如我上面设置的`customSize`属性，类型值是Int型，正确的写法应该是下面这样：

* 正确的写法

```xml
<TextView
    style="@style/TextStyle.Title"
    android:layout_marginTop="16dp"
    app:customSize="@{25}"
    app:customTitle="@{state.title}" />
```

* 常见错误的写法

```xml
<TextView
    style="@style/TextStyle.Title"
    android:layout_marginTop="16dp"
    app:customSize="25"
    app:customTitle="@{state.title}" />
```

上述错误的写法，运行后编译器会报错`AAPT: error: attribute customSize (aka com.xuexiang.databindingsample:customSize) not found.`。

所以当我们写DataBinding的时候，如果出现`AAPT: error: attribute xxx (aka com.aa.bb:xxx) not found.`，十有八九是你赋值漏掉了`"@{}"`。

2. 单例类+@JvmStatic注解

```kotlin
object TitleAdapter {
    @JvmStatic
    @BindingAdapter("customTitle2")
    fun setCustomTitle2(view: TextView, title: String) {
        view.text = "标题4: $title"
    }
}
```

### 3.3 @BindingConversion自定义类型转换

作用：在使用DataBinding的时候，对属性值进行转换，以匹配对应的属性。
定义：方法必须为公共静态（public static）方法，且有且只能有1个参数。

下面我给一个简单的例子：

1.对于User类，`age`的类型是Int。

```kotlin
data class User(
    val name: String,
    val gender: String? = "男",
    val age: Int = 10,
    val phone: String? = "13124765438",
    val address: String? = null
)
```

2.使用`@BindingAdapter`定义了`age`的类型却是String。

```kotlin
@BindingAdapter(value = ["name", "age"], requireAll = true)
fun TextView.setUserInfo(name: String, age: String) {
    text = "${name}今年${age}岁"
}
```

3.这时候使用DataBinding的时候，👇的`app:age="@{state.user.age}"`会编译报错，提示类型不匹配。

```xml
<TextView
    style="@style/TextStyle.Title"
    android:layout_marginTop="16dp"
    app:name="@{state.user.name}"
    app:age="@{state.user.age}"/>
```

4.这个时候，我们就可以使用`@BindingConversion`自定义类型转换: Int -> String, 这样👆的代码就不会编译出错了。

```kotlin
@BindingConversion
fun int2string(integer: Int) = integer.toString()
```

### 3.4 @{}中表达式使用

1. 常用运算符

* 算术 + - / * %
* 字符串合并 +
* 逻辑 && ||
* 二元 & | ^
* 一元 + - ! ~
* 移位 >> >>> <<
* 比较 == > < >= <=
* 三元 ?:
* Array 访问 \[\]

```xml
<TextView
    android:text="@{@string/app_name +  @string/app_name}"/>
```

```xml
<TextView 
    android:visibility="@{!state.user.phone.empty ? View.VISIBLE : View.GONE}"/>
```

2. 常用转义字符

* 空格:  \&nbsp;
* <小于号:  \&lt;
* \>大于号:  \&gt;
* &与号:	 \&amp;

```xml
<TextView 
    android:visibility="@{!state.user.phone.empty &amp;&amp; state.user.age > 5 ? View.VISIBLE : View.GONE}"/>
```

3. 资源使用

@string @color @drawable @dimen @array

```xml
<TextView
    style="@style/TextStyle.Content"
    android:text="@{@string/user_format(state.user.name, state.user.gender)}"
    android:textColor="@{@color/toast_error_color}"
    android:textSize="@{@dimen/xui_config_size_content_text_phone}" />
```

4. 集合

集合不属于`java.lang*`下, 需要导入全路径。集合使用\[\]进行访问。

```xml
<data>
    <import type="java.util.List"/>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
</data>
```
```xml
<TextView
    android:text="@{`key: key1, value:` + map[`key1`]}" />
```

5. 引用类的静态方法

kotlin中定义静态方法，一定要在方法上加上`@JvmStatic`，否则将无法成功引用。

(1) 定义方法
```kotlin
object AppUtils {

    @JvmStatic
    fun getAppInfo(context: Context?) =
        context?.let {
            "packageName: ${it.packageName}, \nversionName: ${
                it.packageManager.getPackageInfo(
                    it.packageName,
                    0
                ).versionName
            }"
        }
}
```
(2) 导入方法所在类路径

```xml
<import type="com.xuexiang.databindingsample.utils.AppUtils"/>
```

(3) 引用方法

```xml
<TextView
    android:text="@{AppUtils.getAppInfo(context)}"/>
```

6. 空值合并运算符

空值合并运算符 ?? 会取第一个不为 null 的值作为返回值。

```xml
<TextView
    android:text="@{`地址：` + (state.user.address ?? `默认地址`)}"/>
```
等价于

```xml
<TextView
    android:text="@{state.user.address != null ?  state.user.address : `默认地址`)}"/>
```

### 5.include 和 viewStub

在主布局文件中将相应的变量传递给 include 布局，需使用自定义的 bind 命名空间将变量传递给 （include/ViewStub）， 从而使两个布局文件之间共享同一个变量。

例如，在include中定义的变量id是：<variable name="user" type="...User"/>, 那么就使用 app:user="@{state.user}" 来绑定数据，与variable定义的name保持一致。

1. include

```xml
<include
    layout="@layout/include_user_info"
    app:user="@{state.user}" />
```

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable
            name="user"
            type="com.xuexiang.databindingsample.fragment.basic.model.User" />

    </data>

    <TextView
        style="@style/TextStyle.Content"
        android:userInfo="@{user}" />
</layout>
```

2. viewStub

```xml
<ViewStub
    android:id="@+id/user_info"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="16dp"
    android:layout="@layout/viewstub_user_info"
    app:info="@{state.user}" />
```

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable
            name="info"
            type="com.xuexiang.databindingsample.fragment.basic.model.User" />

    </data>

    <TextView
        style="@style/TextStyle.Content"
        android:userInfo="@{info}" />
</layout>
```

