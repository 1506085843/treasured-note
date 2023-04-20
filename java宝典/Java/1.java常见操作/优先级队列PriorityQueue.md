[TOC]

# 前言

相较于普通先进先出队列来说，优先级队列会根据优先级进行由高到低排序，出队时优先级高的先出队。

# 一、普通队列对比优先级队列：

**1.普通队列：**

```java
import java.util.LinkedList;
import java.util.Queue;

public class MainTest {
public static void main(String[] args) {    
	    Queue<Integer> queue = new LinkedList<>();
        queue.offer(0);
        queue.offer(2);
        queue.offer(5);
        queue.offer(3);
        queue.offer(6);
        queue.offer(1);
        queue.offer(4);
        queue.offer(0);

        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
	}
}
```

**输出：**

```java
0
2
5
3
6
1
4
0
```

**2.优先级队列:**

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class MainTest {
public static void main(String[] args) {   
        Queue<Integer> p = new PriorityQueue<>();
        p.offer(0);
        p.offer(2);
        p.offer(5);
        p.offer(3);
        p.offer(6);
        p.offer(1);
        p.offer(4);
        p.offer(0);

        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
   }
}
```

**输出：**

```java
0
0
1
2
3
4
5
6
```

**总结：**
1.普通优先级队列先进先出。优先级队列会根据优先级进行排序，优先级高的先出队；
2.对于数字类型的优先级队列，默认数字越小优先级越高；
3.对于字符串类型的优先级对列，默认安照ASCII码位置，位置越小优先级越高，即优先级：字符0到9 >大写字符A到Z > 小写字符a到z    如果字符串首字符一样则依次比较后面的字符判断优先级。

# 二、逆序优先级队列

默认的数字类型优先级队列数字越小优先级越高，字符串类型的优先级对列ASCII码位置越小优先级越高。有时候我们需要数字越大优先级越高或者ASCII码位置越大优先级越高，所以需要改变默认的优先级。

**逆序优先级队列：**

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class MainTest {
public static void main(String[] args) {   
        Queue<Integer> p = new PriorityQueue<>(Collections.reverseOrder());
        p.offer(0);
        p.offer(2);
        p.offer(5);
        p.offer(3);
        p.offer(6);
        p.offer(1);
        p.offer(4);
        p.offer(0);

        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
        System.out.println(p.poll());
   }
}
```

**输出：**
```java
6
5
4
3
2
1
0
0
```



# 三、自定义优先级队列的优先级
优先级队列里根据每个学生的平均分降序排序，即平均分越高优先级越高，越先出队列

**学生类Student：**

```java
import java.util.List;

public class Student  {

    public String name;//姓名
    public List<Integer> score;//分数

    public Student (String name ,List<Integer> score){
        this.name=name;
        this.score=score;
    }

    public Student (){}

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Integer> getScore() {
        return score;
    }

    public void setScore(List<Integer> score) {
        this.score = score;
    }

    @Override
    public String toString() {
        int avg= score.stream().reduce(0, Integer::sum)/score.size();
        return "name:"+name+",score:"+score+"，平均分："+avg;
    }
}
```

**测试：**

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class MainTest {
public static void main(String[] args) {   
        Student student1 = new Student("小明", Arrays.asList(77, 56, 98));
        Student student2 = new Student("小王", Arrays.asList(95, 62, 80));
        Student student3 = new Student("小红", Arrays.asList(82, 69, 73));
        Student student4 = new Student("小刘", Arrays.asList(90, 86, 74));

        Queue<Student> q = new PriorityQueue<>((a, b) -> {
            //计算每个学生分数的平均分，降序排序（即平均分越高优先级越高，越先出队列）
            List<Integer> lisaA = a.getScore();
            List<Integer> lisaB = b.getScore();
            int avgA = lisaA.stream().reduce(0, Integer::sum) / lisaA.size();
            int avgB = lisaB.stream().reduce(0, Integer::sum) / lisaB.size();
            return avgB - avgA;
        });

        q.offer(student1);
        q.offer(student2);
        q.offer(student3);
        q.offer(student4);

        System.out.println(q.poll().toString());
        System.out.println(q.poll().toString());
        System.out.println(q.poll().toString());
        System.out.println(q.poll().toString());
    }
}
```

输出：

```java
name:小刘,score:[90, 86, 74]，平均分：83
name:小王,score:[95, 62, 80]，平均分：79
name:小明,score:[77, 56, 98]，平均分：77
name:小红,score:[82, 69, 73]，平均分：74
```

---
参考：[Priority Queue in Reverse Order in Java](https://www.geeksforgeeks.org/priority-queue-in-reverse-order-in-java/)