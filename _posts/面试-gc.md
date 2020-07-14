准备阶段初始值



GC

回收算法

标记-清除（Mark-Sweep）算法

分为标记和清楚两个阶段：

- 标记阶段 把所有活动对象都做上标记
- 清楚阶段把没有非活动对象回收。

优点



缺点

- 碎片化

复制算法(Copying)

分2个内存区域from和to

- 2个等大空间from,to
- 标记的from的活动对象复制到to
- from空间和to空间互换

- to空间清空

优点

- 可实现高速分配
- 不会发生碎片化
- 与缓存兼容

缺点

- 堆的使用效率低

标记-压缩算法（Mark-Compact）

- 标记阶段 把所有活动对象都做上标记
- 整理阶段 把活动对象压缩，清除非活动对象

优点 

- 没有碎片，节省空间

缺点

- 压缩花费计算成本

<https://www.cnblogs.com/haitaofeiyang/p/7776904.html>





<https://www.cnblogs.com/haitaofeiyang/p/7811311.html>

新生代收集器

Serial（串行收集器）

- 运行方式：单线程

- 算法 ：复制算法

- 应用场景：内存小（10M-200M）短时间完成垃圾收集，不频繁发生

- 优势：简单高效，限定CPU环境，效率高。

- 设置参数：-XX:+UseSerialGC（启用命令）

ParNew（多线程收集器）

- 运行方式：多线程

- 算法：复制算法

- 应用场景：大内存，多CPU
- 优势：相对缩短停顿时间

- 设置参数：-XX:+UseConcMarkSweepGC()   -XX:+UseParNewGC(启用命令)  -XX:ParallelGCThreads(设置线程数)

ParallelScavenge（）

- 运行方式：并行多线程

- 算法：复制算法

- 应用场景：大内存，多CPU，执行批量处理任务、订单处理、科学计算等；
- 优势：相对缩短停顿时间，提高吞吐量

- 设置参数：-XX:+MaxGCPauseMillis(最大停顿时间)   -XX:GCTimeRatio(垃圾收集时间占比)

老年代收集器

SerialOld

- 运行方式：单线程

- 算法 ：标记-整理

ParallelOld

- 运行方式：多线程

- 算法 ：标记-整理

CMS

- 运行方式：多线程

- 算法 ：标记-清除

> 运行过程
>
> > - 初始标记（initial mark）（停顿）标记GC Root Tracing过程
> >
> > - 并发标记（concurrent mark）（用户线程和收集线程同时运行）运行中产生的对象，造成浮动垃圾
> >
> > - 重新标记(remark)(停顿)
> > - 并发清除（concurrent sweep）(用户线程和收集线程同时运行)回收垃圾对象

综合收集器

G1

逻辑分代收集器

G1对java堆划分成独立的域(region)(把堆分片)，由一个优先列表统计Region的优先值，优先回收价值最大的Region