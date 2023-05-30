[TOC]


# 前言
一般的代码中经常会遇到将一个对象的多个属性值赋给另一个对象的属性，就像这样：

```java
Student student = new Student("张三", 1, Arrays.asList(80, 90));
Info info = new Info();
info.setName(student.getName());
info.setScore(student.getScore());
```

当有多个属性需要赋给另一个对象时，将不得不写很多个set/get ,这就有点繁琐，这个时候就可以使用 spring 或者 apache的 BeanUtils 工具类来方便的实现了。

# 一、准备

Student类：

```jav
package com.hai.tang.model;

import java.util.List;

public class Student {
    public String name;

    public int id;

    public List<Integer> score;

    public Address address;

    public Student() {}

    public Student(String name,int id,List<Integer> score,Address address) {
        this.name = name;
        this.id = id;
        this.score = score;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public List<Integer> getScore() {
        return score;
    }

    public void setScore(List<Integer> score) {
        this.score = score;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

Address类：

```java
package com.hai.tang.model;

public class Address {
    public String type;
    public String value;

    public Address(String type, String value) {
        this.type = type;
        this.value = value;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "type:" + type + ",value:" + value;
    }
}
```


# 一、使用apache 的 BeanUtil 

## 1.maven坐标
```java
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.9.4</version>
        </dependency>
```

## 2.使用

```java
import com.hai.tang.model.Address;
import com.hai.tang.model.Student;
import org.apache.commons.beanutils.BeanUtils;
import java.util.Arrays;

public class MainServer {
    public static void main(String[] args) throws Exception {
        Student student = new Student("张三", 1, Arrays.asList(80, 90), new Address("address", "test"));
        Student studentNew = new Student();

        //将student复制给studentNew
        BeanUtils.copyProperties(studentNew, student);

        //测试,两个对象的内存地址是否一样
        System.out.println(student == studentNew);
        //测试,修改属性看另一个会不会变
        student.setAddress(new Address("aaa", "111"));
        System.out.println(studentNew.getAddress());
        System.out.println(student.getAddress());
    }
}
```

输出：

```java
false
type:address,value:test
type:aaa,value:111
```


# 二、使用下面 Spring 的 BeanUtil

## 1.maven坐标

```java
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.3.27</version>
        </dependency>
```

## 2.使用

```java
import com.hai.tang.model.Address;
import com.hai.tang.model.Student;
import org.springframework.beans.BeanUtils;
import java.util.Arrays;

public class MainServer {
    public static void main(String[] args) throws Exception {
        Student student = new Student("张三", 1, Arrays.asList(80, 90), new Address("address", "test"));
        Student studentNew = new Student();

        //将student复制给studentNew
        BeanUtils.copyProperties(student,studentNew);

        //测试,两个对象的内存地址是否一样
        System.out.println(student == studentNew);
        //测试,修改属性看另一个会不会变
        student.setAddress(new Address("aaa", "111"));
        System.out.println(studentNew.getAddress());
        System.out.println(student.getAddress());
    }
}
```

输出：

```java
false
type:address,value:test
type:aaa,value:111
```

# 三、说明
1.apache 和 Spring 的 BeanUtils.copyProperties 方法的两个形参是反的，需要注意下。

```java
//apache 的 BeanUtil,将student复制给studentNew
BeanUtils.copyProperties(studentNew, student);

//Spring  的 BeanUtil,将student复制给studentNew
BeanUtils.copyProperties(student,studentNew);
    
```

2.根据上面 apache 或 Spring 的 BeanUtil 的输出你会发现两个对象的哈希码不相等，引用对象的地址也不相同，而且对包装对象的操做都是互不影响，简单测试下能够看到 BeanUtils 实现了深度拷贝的效果。 但是注意，我在实际项目中测试并不是深拷贝，修改了引用对象的值，另一个可能是会改变的。所以如果拷贝后又修改了成员变量的值那这个时候要注意另一个可能也会改变！如果你对深拷贝和浅拷贝感兴趣可以看下我这篇文章：[java浅拷贝与深拷贝-clone](https://blog.csdn.net/qq_33697094/article/details/119955786)

3.另外由于apache 的 BeanUtil 源码中采用反射机制和封装，其性能有一点慢，因此更推荐使用 Spring 的 BeanUtil。