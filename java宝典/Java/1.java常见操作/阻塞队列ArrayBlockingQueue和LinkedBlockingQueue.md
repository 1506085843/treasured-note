

[TOC]

### 一、BlockingQueue阻塞队列接口介绍
阻塞队列多用于多线程间数据的传递，比如生产者与消费者，A线程往阻塞队列中加数据，B线程往阻塞队列里取数据。所谓阻塞也就是如果队列满了就等一等阻塞在那里，直到有数据再往队列里放或取。阻塞队列多用于多线程处理数据。

实现了公共接口BlockingQueue的阻塞队列有：
**ArrayBlockingQueue，LinkedBlockingDeque，DelayQueue，，LinkedTransferQueue，PriorityBlockingQueue，SynchronousQueue 。**

其中ArrayBlockingQueue，LinkedBlockingDeque比较常用。

**添加元素：**
BlockingQueue 提供了以下方法用于添加元素
| 方法                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| add()                                   | 如果插入成功则返回 true，否则抛出 IllegalStateException 异常 |
| put()                                   | 将指定的元素插入队列，如果队列满了，那么会阻塞直到有空间插入 |
| offer()                                 | 如果插入成功则返回 true，否则返回 false                      |
| offer(E e, long timeout, TimeUnit unit) | 尝试将元素插入队列，如果队列已满，那么会阻塞直到有空间插入   |

**获取元素：**
BlockingQueue 提供了以下方法用于获取元素
| 方法                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| take()                            | 获取队列的头部元素并将其删除，如果队列为空，则阻塞并等待元素变为可用 |
| poll(long timeout, TimeUnit unit) | 检索并删除队列的头部，如有必要，等待指定的等待时间以使元素可用，如果超时，则返回 null |


### 二、ArrayBlockingQueue阻塞队列
**ArrayBlockingQueue：** ArrayBlockingQueue是一个由数组支持的有界阻塞队列。其队列创建时必须指定队列大小，一旦创建，我们就不能增加或缩小队列的大小。如果我们尝试将一个元素插入到一个满了的队列中，那么它将导致操作阻塞。同样，如果我们尝试从空队列中取出一个元素，那么操作也会被阻塞。ArrayBlockingQueue采用FIFO （先进先出）顺序。队列头部或前面的元素是该队列中所有元素中最旧的元素。该队列尾部的元素是该队列所有元素的最新元素。新元素总是插入到队列的末尾或末尾，检索操作获取队列头的元素。

**ArrayBlockingQueue示例：**
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class MainTest {
    public static void main(String[] args) {
        BlockingQueue<Integer> p = new ArrayBlockingQueue<>(3);
        //开一个线程作为生产者
        new Thread(() -> {
            try {
                p.put(0);
                p.put(2);
                p.put(5);
                p.put(6);
                p.put(7);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        //开一个线程作为消费者
        new Thread(() -> {
            try {
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        
      }
}
```
**输出：**

```java
0
2
5
6
7
```

### 三、LinkedBlockingQueue 阻塞队列
**LinkedBlockingQueue :** LinkedBlockingQueue是一个由链表节点支持的可选有界BlockingQueue。创建LinkedBlockingQueue队列时即可以指定队列初始大小，也可以不指定，如果不指定，容量等于Integer.MAX_VALUE就是无界限队列。LinkedBlockingQueue采用FIFO （先进先出）顺序，队列头部或前面的元素是该队列中所有元素中最旧的元素。该队列尾部的元素是该队列所有元素的最新元素。新元素总是插入到队列的末尾或末尾，检索操作获取队列头的元素。链表队列LinkedBlockingQueue 通常比基于数组的ArrayBlockingQueue队列具有更高的吞吐量，但在大多数并发应用程序中的可预测性能较差。

**LinkedBlockingQueue示例：**

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class MainServer {
    public static void main(String[] args) {
    BlockingQueue<Integer> p = new LinkedBlockingQueue<>();
        //开一个线程作为生产者
        new Thread(() -> {
            try {
                p.put(0);
                p.put(2);
                p.put(5);
                p.put(6);
                p.put(7);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        //开一个线程作为消费者
        new Thread(() -> {
            try {
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
                System.out.println(p.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        
      }
   }
```
**输出：**

```java
0
2
5
6
7
```

### 四、ArrayBlockingQueue 和 LinkedBlockingQueue 的区别

| ArrayBlockingQueue                                         | LinkedBlockingQueue                                          |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| ArrayBlockingQueue是有界的，它的大小在创建后永远不会改变。 | LinkedBlockingQueue是可选有界的，你可以选择一个上限。如果未指定上限，默认使用Integer.MAX_VALUE作为上限。 |
| 吞吐量低于LinkedBlockingQueue                              | 吞吐量高于ArrayBlockingQueue                                 |
| 使用单锁双条件算法。生产者和消费者共享一个锁。             | 两个锁队列算法，并且有两个锁条件 putLock 和 takeLock 分别用于从队列中插入和删除元素。 |
| ArrayBlockingQueue 始终持有一个对象数组                    | LinkedBlockingQueue 是一个带有三个对象字段的对象的链表节点。 |

**总结：**
如果不需要阻塞队列，优先选择ConcurrentLinkedQueue；如果需要阻塞队列，队列大小固定优先选择ArrayBlockingQueue，队列大小不固定优先选择LinkedBlockingQueue；如果需要对队列进行排序，选择PriorityBlockingQueue；如果需要一个快速交换的队列，选择SynchronousQueue；如果需要对队列中的元素进行延时操作，则选择DelayQueue。

-----

**拓展：** ArrayBlockingQueue 和LinkedBlockingQueue都是加锁来实现的线程安全阻塞队列。与ArrayBlockingQueue 和LinkedBlockingQueue不同，采用 CAS操作实现的ConcurrentLinkedQueue并发队列也能实现高吞吐量。

参考：
[BlockingQueue比较小结](https://blog.csdn.net/stoger/article/details/6662037)