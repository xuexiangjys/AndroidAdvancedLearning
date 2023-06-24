# RecyclerView性能优化之异步预加载

## 前言

首先需要强调的是，这篇文章是对我之前写的[《RecyclerView的性能优化》](https://juejin.cn/post/7164032795310817294)文章的补充，建议大家先读完这篇文章后再来看这篇文章，味道更佳。

当时由于篇幅的原因，并没有深入展开讲解，于是有很多感兴趣的朋友纷纷留言表示：能不能结合相关的示例代码讲解一下到底如何实现？那么今天我就结合之前讲的如何优化`onCreateViewHolder`的加载时间，讲一讲如何实现`onCreateViewHolder`的异步预加载，希望能给你带来启发。

## 分析

之前我们讲过，在优化`onCreateViewHolder`方法的时候，可以降低item的布局层级，可以减少界面创建的渲染时间，其本质就是降低view的inflate时间。因为`onCreateViewHolder`最大的耗时部分，就是view的inflate。相信读过`LayoutInflater.inflate`源码的人都知道，这部分的代码是同步操作，并且涉及到大量的文件IO的操作以及锁操作，通常来说这部分的代码快的也需要几毫秒，慢的可能需要几十毫秒乃至上百毫秒也是很有可能的。 如果真到了每个ItemView的inflate需要花上上百毫秒的话，那么在大数据量的RecyclerView进行快速上下滑动的时候，就必然会导致界面的滑动卡顿、不流畅。

那么如何你的程序里真的有这样一个列表，它的每个ItemView都需要花上上百毫秒的时间去inflate的话，你该怎么做？

* 首先就是对布局进行优化，降低item的布局层级。但这点的优化往往是微乎其微的。
* 其次可能就是想办法让设计师重新设计，将布局中的某些内容删除或者折叠了，对暂不展示的内容使用ViewStub进行延迟加载。不过说实在话，你既然有能力让设计师重新设计的话，还干个球的开发啊，直接当项目经理不香吗？
* 最后你可能会考虑不用xml写布局，改为使用代码自己一个一个new布局。话说回来了，一个使用xml加载的布局都要花上上百毫秒的布局，可能xml都快上千行下去了，你确定要自己一个一个new下去？

以上的方式，都是建立在列表布局可以修改的情况下，如果我们使用的列表布局是第三方已经提供好的呢？（例如广告SDK等）

那么有没有什么办法既可以不用修改当前的xml布局，又可以极大地缩短布局的加载时间呢？毫无疑问，布局异步加载将为你打开新的世界。

## 原理

> Google官方很早就发现了XML布局加载的性能问题，于是在androidx中提供了异步加载工具AsyncLayoutInflater。其本质就是开了一个长期等待的异步线程，在子线程中inflate view，然后把加载好的view通过接口抛出去，完成view的加载。

一般来说，对于复杂的列表，往往都对应了复杂的数据，而这复杂的数据往往又是通过服务器获取而来。所以一般来说，一个列表在加载前，往往先需要访问服务器获取数据，然后再刷新列表显示，而这访问服务器的时间大约也在300ms～1000ms之间。很多开发人员对这段时间往往没有加以利用，只是加上一个loading动画了事。

其实对于这一段事务真空的时间窗口，我们可以提前进行列表的ItemView的加载，这样等数据请求下来刷新列表的时候，我们`onCreateViewHolder`的时候就可以直接到已经事先预加载好的View缓存池中直接获取View传到ViewHolder中使用，这样`onCreateViewHolder`的创建时间几乎耗时为0，从而极大地提升了列表的加载和渲染速度。详细的流程可以参见下图：

![](https://s1.ax1x.com/2023/06/24/pCtDDyR.png)

## 实现

上面我简单地讲解了一下原理，下一步就是考虑如何实现这样的效果了。

### 预加载缓存池

首先在预加载前，我们需要先创建一个缓存池来存储预加载的View对象。

这里我选择使用`SparseArray`进行存储，key是Int型，存放布局资源的layoutId，value是Object型，存放的是这类布局加载View的集合。

这里的集合类型我选择的是LinkedList，因为我们的缓存需要频繁的添加和删除操作，并且LinkedList实现了Deque接口，具备先入先出的能力。

这里View的引用我选择的是软引用SoftReference，之所以不采用WeakReference, 目的就是希望缓存能多存在一段时间，避免内存的频繁释放和回收造成内存的抖动。

```java
private static class ViewCache {

    private final SparseArray<LinkedList<SoftReference<View>>> mViewPools = new SparseArray<>();

    @NonNull
    public LinkedList<SoftReference<View>> getViewPool(int layoutId) {
        LinkedList<SoftReference<View>> views = mViewPools.get(layoutId);
        if (views == null) {
            views = new LinkedList<>();
            mViewPools.put(layoutId, views);
        }
        return views;
    }

    public int getViewPoolAvailableCount(int layoutId) {
        LinkedList<SoftReference<View>> views = getViewPool(layoutId);
        Iterator<SoftReference<View>> it = views.iterator();
        int count = 0;
        while (it.hasNext()) {
            if (it.next().get() != null) {
                count++;
            } else {
                it.remove();
            }
        }
        return count;
    }

    public void putView(int layoutId, View view) {
        if (view == null) {
            return;
        }
        getViewPool(layoutId).offer(new SoftReference<>(view));
    }

    @Nullable
    public View getView(int layoutId) {
        return getViewFromPool(getViewPool(layoutId));
    }

    private View getViewFromPool(@NonNull LinkedList<SoftReference<View>> views) {
        if (views.isEmpty()) {
            return null;
        }
        View target = views.pop().get();
        if (target == null) {
            return getViewFromPool(views);
        }
        return target;
    }
}
```

从`getViewFromPool`方法我们可以看出，这里对于ViewCache来说，每次取出一个缓存View使用的是`pop`方法，我们都会将它从Pool中移除。

### 布局加载者

因为view的加载方法，涉及到三个参数: 资源Id-resourceId, 父布局-root和是否添加到根布局-attachToRoot。

```java
public View inflate(int resource, ViewGroup root, boolean attachToRoot) {
    
}
```

这里在`onCreateViewHolder`方法中attachToRoot恒为false，因此异步布局加载只需要前面两个参数以及一个回调接口即可，即如下的定义：

```java
public interface ILayoutInflater {
    /**
     * 异步加载View
     *
     * @param parent   父布局
     * @param layoutId 布局资源id
     * @param callback 加载回调
     */
    void asyncInflateView(@NonNull ViewGroup parent, int layoutId, InflateCallback callback);
    /**
     * 同步加载View
     *
     * @param parent   父布局
     * @param layoutId 布局资源id
     * @return 加载的View
     */
    View inflateView(@NonNull ViewGroup parent, int layoutId);
}

public interface InflateCallback {

    void onInflateFinished(int layoutId, View view);
}
```

至于接口实现的话，就直接使用Google官方提供的异步加载工具AsyncLayoutInflater来实现。

```java
public class DefaultLayoutInflater implements PreInflateHelper.ILayoutInflater {

    private AsyncLayoutInflater mInflater;

    private DefaultLayoutInflater() {}

    private static final class InstanceHolder {
        static final DefaultLayoutInflater sInstance = new DefaultLayoutInflater();
    }

    public static DefaultLayoutInflater get() {
        return InstanceHolder.sInstance;
    }

    @Override
    public void asyncInflateView(@NonNull ViewGroup parent, int layoutId, PreInflateHelper.InflateCallback callback) {
        if (mInflater == null) {
            Context context = parent.getContext();
            mInflater = new AsyncLayoutInflater(new ContextThemeWrapper(context.getApplicationContext(), context.getTheme()));
        }
        mInflater.inflate(layoutId, parent, (view, resId, parent1) -> {
            if (callback != null) {
                callback.onInflateFinished(resId, view);
            }
        });
    }

    @Override
    public View inflateView(@NonNull ViewGroup parent, int layoutId) {
        return InflateUtils.getInflateView(parent, layoutId);
    }
}
```

### 预加载辅助类

有了预加载缓存池ViewCache和异步加载能力的提供者IAsyncInflater，下面就是来协调这两者进行合作，完成布局的预加载和View的读取。

首先需要定义的是根据ViewGroup和layoutId获取View的方法，提供给Adapter的`onCreateViewHolder`方法使用。

* 首先我们需要去ViewCache中去取是否已有预加载好的view供我们使用。如果有则取出，并进行一次预加载补充给ViewCache。
* 如果没有，就只能同步加载布局了。

```java
public View getView(@NonNull ViewGroup parent, int layoutId, int maxCount) {
    View view = mViewCache.getView(layoutId);
    if (view != null) {
        UILog.dTag(TAG, "get view from cache!");
        preloadOnce(parent, layoutId, maxCount);
        return view;
    }
    return mLayoutInflater.inflateView(parent, layoutId);
}
```

对于预加载布局，并加入缓存的方法实现。

* 首先我们需要去ViewCache查询当前可用缓存的数量，如果可用缓存的数量大于等于最大数量，即不需要进行预加载。
* 对于需要预加载的，需要计算预加载的数量，如果当前没有强制执行的次数，就直接按剩余最大数量进行加载，否则取强制执行次数和剩余最大数量的最小值进行加载。
* 对于预加载完毕获取的View，直接加入到ViewCache中。

```java
public void preload(@NonNull ViewGroup parent, int layoutId, int maxCount, int forcePreCount) {
    int viewsAvailableCount = mViewCache.getViewPoolAvailableCount(layoutId);
    if (viewsAvailableCount >= maxCount) {
        return;
    }
    int needPreloadCount = maxCount - viewsAvailableCount;
    if (forcePreCount > 0) {
        needPreloadCount = Math.min(forcePreCount, needPreloadCount);
    }
    for (int i = 0; i < needPreloadCount; i++) {
        // 异步加载View
        mLayoutInflater.asyncInflateView(parent, layoutId, new InflateCallback() {
            @Override
            public void onInflateFinished(int layoutId, View view) {
                mViewCache.putView(layoutId, view);
            }
        });
    }
}
```

### Adapter中执行预加载

有了预加载辅助类PreInflateHelper，下面我们只需要直接调用它的`preload`方法和`getView`方法即可。这里需要注意的是，ViewHolder中ItemView的ViewGroup就是RecyclerView它本身，所以Adapter的构造方法需要传入RecyclerView供预加载辅助类进行预加载。

```java
public class OptimizeListAdapter extends MockLongTimeLoadListAdapter {
    private static final class InstanceHolder {
        static final PreInflateHelper sInstance = new PreInflateHelper();
    }
    
    public static PreInflateHelper getInflateHelper() {
        return OptimizeListAdapter.InstanceHolder.sInstance;
    }

    public OptimizeListAdapter(RecyclerView recyclerView) {
        getInflateHelper().preload(recyclerView, getItemLayoutId(0));
    }

    @Override
    protected View inflateView(@NonNull ViewGroup parent, int layoutId) {
        return getInflateHelper().getView(parent, layoutId);
    }
}
```

## 对比实验






