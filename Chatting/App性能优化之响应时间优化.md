# App性能优化之响应时间优化

速度优化的核心：空间 -> 时间

* 1.能读缓存读缓存（缓存优先）
* 2.能复用绝不新建（减少新建）
* 3.能不做的尽量不做（减少任务）
* 4.具体问题具体分析（必须做的能提前做就提前做，不必须做的延后做）

## 任务执行

* 业务/任务的整合/拆分，梳理流程
* 串行 -> 并行, 同步 -> 异步
* 执行顺序按优先级调整
* 延迟执行
* 空闲执行，如:`IdleHandler`

## 资源加载

* 懒加载
* 部分加载（分段加载）
* 预加载（数据、布局页面等）

## 数据结构

* 数据结构优化（空间大小、读取速度、复用性、扩展性），例如：ArrayList和LinkedList、HashMap和SparseArray。
* 数据缓存（内存缓存、磁盘缓存、网络缓存），分段缓存。这里可以参考glide.
* 锁优化（减少过度锁，避免死锁），悲观锁/乐观锁。
* 内存优化，避免内存抖动，频繁GC（尤其关注bitmap）

## 线程/IO

* 线程优化（统一、优先级调度、任务特性）
* IO优化（网络IO和磁盘IO），核心是减少IO次数
  * 网络：请求合并，请求链路优化，请求体优化，系列化和反序列化优化，请求复用等。
  * 磁盘：文件随机读写、SharePreference读写等（例如对于读多写少的，可使用内存缓存）
* log优化（循环中的log打印，不必要的log打印，log等级）

## 页面渲染

* 降低布局层级、减少嵌套、避免过度渲染（背景）(merge，ConstraintLayout)
* 页面复用(include)
* 页面懒加载
* 布局延迟加载(ViewStub)
* inflate优化（布局预加载+异步加载，动态new控件/X2C）
* 动画优化（注意动画的执行耗时和内存占用）
* 自定义view优化（减少onDraw、onLayout、onMeasure的对象创建和执行耗时）
* bitmap和canvas优化（bitmap大小、质量、压缩、复用；canvas复用，clipRect）
* RecycleView优化（减少刷新次数，缓存复用）

## 推荐工具

* systrace、[Perfetto](https://ui.perfetto.dev/#!/) 、Profile
* [performance](https://github.com/xanderwang/performance)
* [LeakCanary](https://github.com/square/leakcanary)
* [DoKit](https://github.com/didi/DoKit)
