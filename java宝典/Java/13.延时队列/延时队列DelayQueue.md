应用场景：比如放入队列后延时1分钟发短信，任务延时1分钟再次执行等


**实现Delayed接口**
```java
package com.xu.tao.model;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class Calculatenumber implements Delayed {
    private final long expire;  //到期时间
    private final int date;  //数据

    public int getDate(){
        return date;
    }

//延时时间，单位为毫秒
    public Calculatenumber(long delay, int date) {
        this.date = date;
        expire = System.currentTimeMillis() + delay;    //到期时间 = 当前时间+延迟时间
    }

    /**
     * 用于延迟队列内部比较排序   当前时间的延迟时间 - 比较对象的延迟时间
     * @param o
     * @return
     */
    @Override
    public int compareTo(Delayed o) {
        return (int) (this.getDelay(TimeUnit.MILLISECONDS) -o.getDelay(TimeUnit.MILLISECONDS));
    }

    /**
     * 需要实现的接口，获得延迟时间   用过期时间-当前时间
     * @param unit
     * @return
     */
    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(this.expire - System.currentTimeMillis() , TimeUnit.MILLISECONDS);
    }
}
```
**测试一：**
**使用延时队列创建5个延时任务，每个任务有不同的延时时间。（线程不会释放）**

```java
    public static void main(String[] args) throws IOException {
        DelayQueue<Calculatenumber> d = new DelayQueue<Calculatenumber>();
        //开一个线程作为生产者
        new Thread(() -> {
            LocalDateTime localDateTime = LocalDateTime.now();
            DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
            System.out.println(localDateTime.format(f1));
            System.out.println("开始放入1");
            d.put(new Calculatenumber(3000, 1));
            System.out.println("开始放入2");
            d.put(new Calculatenumber(2000, 2));
            System.out.println("开始放入3");
            d.put(new Calculatenumber(1000, 3));
            System.out.println("开始放入4");
            d.put(new Calculatenumber(5000, 4));
            System.out.println("开始放入5");
            d.put(new Calculatenumber(500, 5));
        }).start();

        //开一个线程作为消费者
        new Thread(() -> {
            try {
                while (true) {
                    DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                    System.out.println("开始消费" + d.take().getDate());
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 开启一个计时调度，延迟 0毫秒（也就是立即开始执行），每秒输出当前时间
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                LocalDateTime localDateTime = LocalDateTime.now();
                DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                System.out.println(localDateTime.format(f1));
            }
        }, 0L, 1000L);
    }
```
**输出：**

```java
2022-01-05 16:27:04
2022-01-05 16:27:04
开始放入1
开始放入2
开始放入3
开始放入4
开始放入5
开始消费5
2022-01-05 16:27:05
开始消费3
2022-01-05 16:27:06
开始消费2
2022-01-05 16:27:07
开始消费1
2022-01-05 16:27:08
2022-01-05 16:27:09
开始消费4
2022-01-05 16:27:10
```

**测试二：**
**使用延时队列创建5个延时任务，每个任务有不同的延时时间。5个延时任务执行结束后关闭线程，释放线程资源**

```java
 public static void main(String[] args) {
        DelayQueue<Calculatenumber> d = new DelayQueue<Calculatenumber>();
        //使用CountDownLatch来等待所有2个线程执行完，计数器初始化为2
        CountDownLatch countDownLatch = new CountDownLatch(2);

        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(2);
        //开一个线程作为生产者，放入5个延时任务
        fixedThreadPool.execute(() -> {
            LocalDateTime localDateTime = LocalDateTime.now();
            DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
            System.out.println(localDateTime.format(f1));
            System.out.println("开始放入1");
            d.put(new Calculatenumber(6000, 1));
            System.out.println("开始放入2");
            d.put(new Calculatenumber(500, 2));
            System.out.println("开始放入3");
            d.put(new Calculatenumber(3000, 3));
            System.out.println("开始放入4");
            d.put(new Calculatenumber(5000, 4));
            System.out.println("开始放入5");
            d.put(new Calculatenumber(4000, 5));
            //当此线程执行完，线程计数器减1
            countDownLatch.countDown();
        });

        //原子类变量，用于记录执行5次延时任务执行完成后消费线程退出
        AtomicInteger num = new AtomicInteger(0);
        //开一个线程作为消费者
        fixedThreadPool.execute(() -> {
            try {
                while (true) {
                    System.out.println("开始消费" + d.take().getDate());
                    num.incrementAndGet();
                    System.out.println("num：" + num.get());
                    if (num.get() == 5) {
                        System.out.println("num是5");
                        countDownLatch.countDown();
                        break;
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 开启一个计时调度，延迟 0毫秒（也就是立即开始执行），每秒输出当前时间
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                LocalDateTime localDateTime = LocalDateTime.now();
                DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                System.out.println(localDateTime.format(f1));
            }
        }, 0L, 1000L);

        //等到线程计数器变为0，也就是确保30个线程都执行完
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //输出最终的num
        System.out.println("延时任务执行结束：" + num);
        //关闭线程池
        fixedThreadPool.shutdown();
        //检测线程状态
        ThreadPoolExecutor tpe = ((ThreadPoolExecutor) fixedThreadPool);
        int activeCount = tpe.getActiveCount();
        int queueSize = tpe.getQueue().size();
        System.out.println("当前排队线程数：" + queueSize);
        System.out.println("当前活动线程数：" + activeCount);
        long completedTaskCount = tpe.getCompletedTaskCount();
        System.out.println("执行完成线程数：" + completedTaskCount);
        long taskCount = tpe.getTaskCount();
        System.out.println("总线程数：" + taskCount);
    }
```

参考：
[Java延时队列DelayQueue的使用](https://my.oschina.net/lujianing/blog/705894)