

# Lock接口

## 简单的AQS介绍

### AQS内的链表队列

```
volatile Node prev;
volatile Node next;
```

### AQS维护一个队列

```
private transient volatile Node head;

private transient volatile Node tail;

private volatile int state;
```

## Unsafe JNI类

```
//让线程去等待
Thread thread=new Thread(
        ()->{
            unsafe.park(false,0L);
            System.out.println("线程一执行");
        }
);
thread.start();
//唤起线程
unsafe.unpark(thread2)；
```





## ReentrantLock（排它锁）

### 非公平锁

> 加锁过程

```
 Lock mainLock = new ReentrantLock(false);
 //添加锁
 mainLock.lock()
 //解锁
 mainLock.unlock();
```



```
 //锁的工具
 private final Sync sync;
 
 public void lock() {
        sync.lock();
 }
```



```
 //具体实现
 final void lock() {
 	 //锁空闲直接获取锁 ->执行 否则 进入队列
     if (compareAndSetState(0, 1))
     	setExclusiveOwnerThread(Thread.currentThread());
     else
		//放入对列     
         acquire(1);
 }
```



```
 //添加队列
public final void acquire(int arg) {
		//尝试执行不成功， 
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```



```
protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
```



```
final boolean nonfairTryAcquire(int acquires) {//acquires =1 
            final Thread current = Thread.currentThread();
            int c = getState();
            //当前锁为空， 获取锁 ->执行
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //当前线程就是 有锁线程 ->执行
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



```
//当前线程添加到队列中
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        //获取当前尾部节点
        Node pred = tail;
        //是否是尾部节点
        if (pred != null) {
			//是尾部节点，新节点->前指针指向尾部节点        
            node.prev = pred;
            //判读是否是尾部节点
            if (compareAndSetTail(pred, node)) {
                //当前尾部节点->后指针指向新节点
                pred.next = node;
                return node;
            }
        }
        //
        enq(node);
        return node;
    }
```



```
//有线程先于当前线程操作
//当前最后节点不是尾部节点
private Node enq(final Node node) {
        //cas插入节点
        for (;;) {
        	
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    //初始化链表
                    tail = head;
            } else {
                //插入节点
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```



```

final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            //提高唤醒效率
            for (;;) {
            	//当前节点的上个节点
                final Node p = node.predecessor();
                //上个节点是否为头节点，且让头节点执行。当前节点晋升为头节点
                if (p == head && tryAcquire(arg)) {
                    //设置head
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //节点是否中断
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```



```
//挂起当前线程
private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;

        node.thread = null;

        // Skip cancelled predecessors
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        // predNext is the apparent node to unsplice. CASes below will
        // fail if not, in which case, we lost race vs another cancel
        // or signal, so no further action is necessary.
        Node predNext = pred.next;

        // Can use unconditional write instead of CAS here.
        // After this atomic step, other Nodes can skip past us.
        // Before, we are free of interference from other threads.
        node.waitStatus = Node.CANCELLED;

        // If we are the tail, remove ourselves.
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
```



```
//线程中断
static void selfInterrupt() {
        Thread.currentThread().interrupt();
    }
```

> 解锁过程 



```
 public void unlock() {
        sync.release(1);
   }
```



```
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```



```
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```



```
private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
        	//唤醒下一个线程
            LockSupport.unpark(s.thread);
    }
```



### 公平锁





# 参考博客

<https://blog.csdn.net/u012152619/article/details/74977570>

<https://www.jianshu.com/p/3204049fbf14>

