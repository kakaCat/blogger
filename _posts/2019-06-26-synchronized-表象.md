# synchronized

锁的实现就是抢夺唯一的条件。

| 锁类型 |            条件             |
| :----- | :-------------------------: |
| 对象锁 |    对象实例（this,指针）    |
| 类锁   | class实例（锁定元空间数据） |

## 对象实例锁例子

```java
public synchronized void methode1(String name){
    System.out.println("Method 1 start "+ name);

    try {
        System.out.println("Method 1 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 1 end "+ name);
}
```

```java
public synchronized void methode2(String name){
    System.out.println("Method 2 start "+ name);

    try {
        System.out.println("Method 2 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 2 end "+ name);
}
```

### 执行一个对象实例（指针）

```
public static void main(String[] args) {
    SynClassDome dome = new SynClassDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();
}
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 1 end dome
Method 2 start dome
Method 2 execute dome
Method 2 end dome
```

### 结论

一个对象实例，多线程调用锁方法

- 有锁方法是串行执行。

### 执行多个对象实例（指针）

```java
public static void main(String[] args) {
    SynClassDome dome = new SynClassDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();

    SynClassDome dome2 = new SynClassDome();
    new Thread(() -> dome2.methode1("dome2")).start();
    new Thread(() -> dome2.methode2("dome2")).start();
}
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 1 start dome2
Method 1 execute dome2
Method 1 end dome2
Method 1 end dome
Method 2 start dome2
Method 2 start dome
Method 2 execute dome
Method 2 execute dome2
Method 2 end dome
Method 2 end dome2
```

### 结论

多个对象实例，多线程调用有锁方法。

- 不同对象实例调用有锁方法是并发的，相互没有影响。
- 但同对象实例的实例调用有锁方法，是串行执行。

### 对象实例锁 和对象实例中的方法

```java
public  void methode3(String name){
    System.out.println("Method 3 start "+ name);

    try {
        System.out.println("Method 3 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 3 end "+ name);
}
public static void methode4(String name){
    System.out.println("Method 4 start "+ name);

    try {
        System.out.println("Method 4 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 4 end "+ name);
}
```

### 执行对象实例中的锁方法和无锁方法

```java
public static void main(String[] args) {
    SynClassDome dome = new SynClassDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode3("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();
}
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 3 start dome
Method 3 execute dome
Method 4 start dome
Method 4 execute dome
Method 3 end dome
Method 4 end dome
Method 1 end dome
```

### 结论

一个对象实例，多线程调用锁方法和无锁方法（静态方法，有实例方法）

- 对象实例调用有锁方法和无锁方法是并发的

## class锁例子 

```java
public synchronized static void methode1(String name){
    System.out.println("Method 1 start "+ name);

    try {
        System.out.println("Method 1 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 1 end "+ name);
}

public synchronized static void methode2(String name){
    System.out.println("Method 2 start "+ name);

    try {
        System.out.println("Method 2 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 2 end "+ name);
}
```

### 执行class实例锁

```java
    public static void main(String[] args) {
        final SynClassDome dome = new SynClassDome();
        new Thread(() -> dome.methode1("dome")).start();
        new Thread(() -> dome.methode2("dome")).start();
    }
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 1 end dome
Method 2 start dome
Method 2 execute dome
Method 2 end dome
```

### 结论

class锁，调用静态锁方法

- 锁方法，串行执行

### 执行多个class实例锁

```java
public static void main(String[] args) {
    final SynClassDome dome = new SynClassDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();
    SynClassDome dome2 = new SynClassDome();
    new Thread(() -> dome2.methode1("dome2")).start();
    new Thread(() -> dome2.methode2("dome2")).start();
}
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 1 end dome
Method 2 start dome2
Method 2 execute dome2
Method 2 end dome2
Method 1 start dome2
Method 1 execute dome2
Method 1 end dome2
Method 2 start dome
Method 2 execute dome
Method 2 end dome
```

### 结论

多个class实例，调用静态锁

- 多个实例锁方法，串行执行。

### class实例锁 和对象实例中的方法

```java
public  void methode3(String name){
    System.out.println("Method 3 start "+ name);

    try {
        System.out.println("Method 3 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 3 end "+ name);
}
public static void methode4(String name){
    System.out.println("Method 4 start "+ name);

    try {
        System.out.println("Method 4 execute "+ name);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 4 end "+ name);
}
```

### 执行class实例中的锁方法和无锁方法

```java
public static void main(String[] args) {
    final SynClassDome dome = new SynClassDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode3("dome")).start();
    new Thread(() -> dome.methode4("dome")).start();
}
```

### 结论

class实例锁，多线程调用锁方法和无锁方法（静态方法，有实例方法）

- 对象实例调用有锁方法和无锁方法是并发的

## 对象实例方法中的锁例子

```
public void methode1(String name){
    System.out.println("Method 1 start "+ name);

    try {
        synchronized(this){
            System.out.println("Method 1 execute "+ name);
            Thread.sleep(4000);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 1 end "+ name);
}

public  void methode2(String name){
    System.out.println("Method 2 start "+ name);

    try {
        synchronized(this){
            System.out.println("Method 2 execute "+ name);
            Thread.sleep(4000);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Method 2 end "+ name);
}
```

### 执行对象实例方法中的锁

```java
public static void main(String[] args) {
    final SynMethodDome dome = new SynMethodDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();
}
```

### 结果

```
Method 1 start dome
Method 2 start dome
Method 2 execute dome
Method 2 end dome
Method 1 execute dome
Method 1 end dome
```

### 结论

对象实例方法中的锁，多线程调用方法中的锁

- 只有加锁的的范围内的，逻辑串行执行

### 执行多个对象实例方法中的锁

```java
public static void main(String[] args) {
    final SynMethodDome dome = new SynMethodDome();
    final SynMethodDome dome2 = new SynMethodDome();
    new Thread(() -> dome.methode1("dome")).start();
    new Thread(() -> dome.methode2("dome")).start();

    new Thread(() -> dome2.methode1("dome2")).start();
    new Thread(() -> dome2.methode2("dome2")).start();

}
```

### 结果

```
Method 1 start dome
Method 1 execute dome
Method 2 start dome
Method 1 start dome2
Method 1 execute dome2
Method 2 start dome2
Method 1 end dome
Method 2 execute dome
Method 2 execute dome2
Method 1 end dome2
Method 2 end dome
Method 2 end dome2
```

### 结论

多对象实例方法中的锁，多线程调用方法中的锁

- 不同对象实例调用有锁方法是并发的，相互没有影响。
- 但同对象实例中只有加锁的的范围内的，逻辑串行执行



## 最终结论

synchronized是非公平锁，谁抢到锁，谁先执行。

| 类型           |          方法加锁          | 静态方法和方法 |
| :------------- | :------------------------: | :------------: |
| 对象实例       | 串行（在同一个调用锁方法） |      并发      |
| 多对象实例     |    并发（对象实例之间）    |      并发      |
| class实例      |            串行            |      并发      |
| 多class实例    |            串行            |      并发      |
| 实例方法块锁   |        加锁范围串行        |      并发      |
| 多实例方法块锁 |        加锁范围串行        |      并发      |

- 静态方法和方法没有加锁，都是并发。
- 加锁范围的逻辑串行执行。



## Java怎么实现sychronized的锁

监视锁(Monitor)是操作系统同步的一个重要概念,在Java中的同步机制也是基于同样的思想.[Monitor](<https://www.programcreek.com/2011/12/monitors-java-synchronization-mechanism/>)

Monitor实现原理是维护一个队列，队列存储了不同线程对象的状态。

JVM模型，可知栈是执行逻辑，值运算的区域，堆是存储对象数据的位置，元空间是存储对象方法，和静态方法

- 当锁定对象实例的方法，调用会被monitor标记。在方法上添加标志位，ACC_SYNCHRONIZED
- 当锁定class实例的方法，调用会被monitor标记。在方法上添加标志位，ACC_STATIC, ACC_SYNCHRONIZED

- 当锁定对象方法中的一段逻辑，调用会被monitor标记。在栈中添加标志位，monitorenter，monitorexit











下面的没写完

| 锁类型   |  线程  | 性能 |
| :------- | :----: | :--: |
| 无锁     | 不阻塞 |  0   |
| 偏向锁   | String |  1   |
| 轻量级锁 | String |  2   |
| 重量级锁 |  JSON  |  3   |

**偏向锁**

偏向锁是Java 6之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。



**轻量级锁**

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(1.6之后加入的)，此时Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。





## 参考

[博客-纯粹的码农](<https://blog.csdn.net/chen77716/article/details/6618779>)

<https://www.cnblogs.com/charlesblc/p/5994162.html>

<https://www.cnblogs.com/mingyao123/p/7424911.html>

[微博 -monitor](https://blog.csdn.net/hello_worldee/article/details/77919549)

<http://www.php.cn/java-article-410323.html>



<https://blog.csdn.net/u012998254/article/details/82558178>



<https://blog.csdn.net/chenmh12/article/details/90234543>

<https://blog.csdn.net/yinwenjie/article/details/83069483>