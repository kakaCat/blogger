# 1.概述

## 1.1 编写目的

本文介绍LUR缓存算法（Least Recently Used）

-  数据使用的时间越近，越不会被淘汰。当缓存空间不足，删除使用时间最远的那条数据。

- **最近使用的页面数据会在未来一段时期内仍然被使用,已经很久没有使用的页面很有可能在未来较长的一段时间内仍然不会被使用**

## 1.2名词解释

## 1.3 参考资料

# 2 LUR

## 2.1LUR的内容

## 2.1.1 LUR 的数据结构

本次现实的结构利用LinkedHashMap

- hash结构
  - 实现缓存数据查找的O(1)

- 双向链表
  - 模拟了队列
  - 近期没有被使用的数据放在队列头



## 2.2 LUR的实现



### 2.2.1 利用LinkedHashMap实现LUR缓存

```
public class LRU<K,V> extends LinkedHashMap<K, V> implements Map<K, V> {

    private static final long serialVersionUID = 1L;

    private int cacheSize;


    public LRU(int initialCapacity,
               float loadFactor,
               boolean accessOrder,int limit) {
        super(initialCapacity, loadFactor, accessOrder);
        if(limit<=0){
            throw new IllegalArgumentException("Illegal load limit: " +
                    limit);
        }
        this.cacheSize=cacheSize;
    }

    /**
     * @description 重写LinkedHashMap中的removeEldestEntry方法，当LRU中元素多余6个时，
     *              删除最不经常使用的元素
     * @author rico
     * @created 2017年5月12日 上午11:32:51
     * @param eldest
     * @return
     * @see java.util.LinkedHashMap#removeEldestEntry(java.util.Map.Entry)
     */
    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {

        return size() > cacheSize ? true : false;

    }
}
```

