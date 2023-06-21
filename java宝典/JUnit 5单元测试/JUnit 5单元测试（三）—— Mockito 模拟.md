[TOC]

# 前言
本文为 JUnit 5单元测试（三）—— Mockito 模拟

想象下面这几种情况你该怎么单元测试：
1.A方法去数据库查询了数据进行了一些处理，该怎么单元测试；
2.在微服务项目中，A方法中调用了远程微服务B方法（或者B方法还没写好），该怎么单元测试；
3.A方法中从 redis 或者 Kafka 消息队列里取了一些数据处理，该怎么单元测试；

可以看到上面几种情况如果仅用断言不能很好的支持单元测试，这时候就可以用 Mockito 来模拟数据进行单元测试了。


# 一、什么是 Mockito
Mockito是一款开源测试库，简称 Mock , 该框架允许在自动化或单元测试中模拟对象。简单来说对于某些不容易构造或者不容易获取的比较复杂的数据/场景，模拟一个虚假的Mock对象来替代真实的对象。

**想象一下这样的情景：**
一个用于和支付提供商（如 支付宝、某银行）通信的 Java类，如果你测试时使用实时支付环境来对信用卡收费相关代码进行测试是很危险的，而且每次运行单元测试时都需要实际连接到支付提供商，这会使单元测试具有不确定性，例如，如果支付提供商由于某种原因关闭了，那就不方便测试了。

如果你的测试数据依赖于外部系统、文件读取时间过长、数据库连接不可靠，或者你不想在每次测试时发送电子邮件，那么 Mock 很有用。

**Mock 一般用于以下情况的单元测试模拟数据：**

- 1.MVC接口验证，比如HTTP接口
- 2.数据库，做单元测试不需要连接数据库
- 3.配置中心、网关等微服务发现治理依赖
- 4.Redis、zookeeper、mq消息队列等第三方中间件
- 5.邮件、log、文件等服务
- 6.对其他服务有依赖的

# 二、引入依赖


```xml
		<!-- 由于mockito 5 开始支持的最低版本是jdk11，这里使用mockito 4的最新版本来支持jdk8及以上-->
		<dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>4.11.0</version>
            <scope>test</scope>
        </dependency>
		 <!-- 用于单元测试中使用@Mock注解时使用@ExtendWith(MockitoExtension.class)-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>4.11.0</version>
            <scope>test</scope>
        </dependency>
```

# 三、创建 mock 实例
假设一个方法是查询数据库返回List集合，现在通过mock 来模拟返回的数据，首先要创建 mock 实例来模拟数据。 

创建 mock 实例有三种方法：调用静态 mock 方法、调用openMocks方法+@Mock 注解、Mockito扩展+@Mock 注解


后文使用到的 Student 类如下：

```java

public class Student {

    public String name;
    public int id;

    public Student(String name, int id) {
        this.name = name;
        this.id = id;
    }

    public String sayHello(String name) {
        return "hello" + name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName(String name,int id) {
        return name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}
```


## 1. 调用静态 mock 方法初始化 mock

在测试方法里使用 MockitoAnnotations.openMocks(this) 来初始化 mock，然后使用 mock 静态方法来模拟一个对象实例:

```java
import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import java.util.List;
import static org.mockito.Mockito.*;

public class MockTest {
    @Test
    public void whenNotUseMockAnnotation_thenCorrect() {
        //使用静态 mock 方法来模拟一个 List 对象
        Student student = mock(Student.class);
		
		//使用student做一些操作
		//......
	}
}
```




## 2. @Mock 注解初始化 mock
除了上面 mock 静态方法来创建模拟对象实例，还可以使用 openMocks 来初始化 mock 然后使用 @Mock 注解来更简单的创建模拟对象实例。

```java
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import java.util.List;
import static org.mockito.Mockito.*;

public class MockTest {
    @Mock
    Student student; //使用 @Mock 注解来模拟一个student对象

    @Test
    public void whenNotUseMockAnnotation_thenCorrect() throws Exception {
		//初始化Mock，（以前低版本的写法是使用initMocks(this)现高版本中改方法已被废弃，转而使用openMocks(this)初始化）
		AutoCloseable closeable = MockitoAnnotations.openMocks(this);
	
		//使用mockedList做一些操作
		//......
		
		//关闭mock
		closeable.close();
	}
}
```


可以把初始化 Mock 和关闭 mock 的代码放到 @BeforeEach 和 @AfterEach 注解的方法中更合适，这样如果你有多个测试方法就不必每个测试方法中都再写一遍初始化和关闭Mock的代码了：

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import java.util.List;
import static org.mockito.Mockito.*;

public class MockTest {
    @Mock
    Student student;

    private AutoCloseable closeable;

    @BeforeEach
    void initService() {
        closeable = MockitoAnnotations.openMocks(this);

    }

    @AfterEach
    void closeService() throws Exception {
        closeable.close();
    }

    @Test
    public void whenNotUseMockAnnotation_thenCorrect() {
		//使用mockedList做一些操作
		//......
	
	}
}
```

## 3. 使用Mockito JUnit 5 扩展来初始化 mock

除了上面两种方式，还有一个用于JUnit 5的 Mockito 扩展库，它初始化 mock 更加简单，一般用这种方式用的比较多，下文的所有示例都将采用这种方式。

先添加如下 mock 扩展依赖：
```xml
		<!-- 用于单元测试中使用@Mock注解时使用@ExtendWith(MockitoExtension.class)-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>4.11.0</version>
            <scope>test</scope>
        </dependency>
```
在测试类上面加上 @ExtendWith(MockitoExtension.class) 注解 ，然后使用 @Mock 注解修饰模拟对象即可：

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.jupiter.MockitoExtension;
import java.util.List;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class MockTest {

    @Mock
    Student student;
    
    @Test
    public void whenNotUseMockAnnotation_thenCorrect() {
		//...
	}
}
```

如果你测试类里有多个测试方法，不想每个测试方法都共享模拟变量，还可以将模拟对象注入到方法参数：

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import java.util.List;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class MockTest {

    @Test
    public void whenNotUseMockAnnotation_thenCorrect(@Mock Student student) {
		//...
	}
}
```




# 四、初始化mock后，mock对象会覆盖掉整个被mock的对象
初始化mock后，mock对象会覆盖掉整个被mock的对象，当你直接调用mock实例对象的方法不会走真实的方法，只会返回默认值（返回null或者空集合，或者0等基本类型的值）。


举个例子：
```java
    @Mock
    Student student ;
    @Mock
    List<String> list ;

    @Test
    public void whenThenCorrect()  {
        student.setId(1);
        System.out.println(student.getId()); //输出0
        list.add("a");
        System.out.println(list.get(0)); //输出null
        System.out.println(list.size()); //输出0
	}
```

所以对于初始化之后的 mock 实例对象是不能直接调用其方法进行返回东西的，要让 mock 实例对象返回东西需要用 when..thenReturn 模拟方法返回值。


# 五、when..thenReturn 模拟方法返回值

## (1) 对于有返回值的方法
when(mock.someMethod(arg1, arg2, ...)).thenReturn(value)用于设置模拟的实例方法返回值，设置后再调用该方法不会运行实例方法里的逻辑，将直接返回模拟的值：

```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect() {
		//设置 student.getName("张三",1)的返回值是"模拟的张三"，后面的代码如果调用 student.getName("张三",1)将直接返回"模拟的张三"，不会去执行 student.getName()里的逻辑
        when(student.getName("张三",1)).thenReturn("模拟的张三");
		//调用student.getName("张三",1)断言为 "张三"
        assertEquals("模拟的张三", student.getName("张三",1));
	}
```

可以在 thenReturn(value1, value2, ...) 里设置连续设定返回值,第一次调用时返回 value1,第二次返回value2：

```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect()  {
        when(student.getName("张三",1)).thenReturn("张三","李四");
        assertEquals("张三", student.getName("张三",1));
        assertEquals("李四", student.getName("张三",1));
	}
```

## (2) 对于无返回值的方法

对于无返回值的方法使用 doNothing().when(mock).someMethod(arg1, arg2, ...)来模拟

```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect()  {
        doNothing().when(student).sayHello("张三");
        student.sayHello("张三");
	}
```



**注意：**
对于 static 、 final 、private修饰的方法和equals()、hashCode()方法， Mockito 无法对其进行when(…).thenReturn(…) 操作。


# 六、参数化匹配器


```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect() {
        when(student.getName("张三",1)).thenReturn("模拟的张三");
        assertEquals("模拟的张三", student.getName("张三",1));
	}
```
在上面 when(mock.someMethod(arg1, arg2, ...)).thenReturn(value) 里，我们所有的参数 arg1、arg2 都是写死的，就像 student.getName("张三",1)这样，这样当调用的时候也要写死了。

我们可以用参数化匹配器来优化下：

## (1) mockito 提供了很多参数匹配器
如：anyInt()、anyString()、anyDouble()、anyList()、anyMap()等
```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect() {
		//使用参数化匹配器 anyString()和 anyInt()
        when(student.getName(anyString(),anyInt())).thenReturn("张三","李四");
        //调用student.getName 随便传入两个参数，断言为 "张三"
        assertEquals("张三", student.getName("aa",12));
		//调用student.getName 随便传入两个参数，断言为 "李四"
        assertEquals("李四", student.getName("bb",12));
	}
```

## (2) 使用参数匹配器时，方法里所有参数都应使用匹配器。 

例如下面的写法就是错的：
```java
	when(student.getName(anyString(),1)).thenReturn("张三","李四");
```
如果要为参数使用特定值，则可以使用eq()方法:
```java
    @Mock
    Student student;

    @Test
    public void whenThenCorrect() {
		//使用参数化匹配器 anyString()和 anyInt()
        when(student.getName(anyString(),eq(1))).thenReturn("张三","李四");
        //调用student.getName 第一个参数随便传入，第二个参数要传1。断言为 "张三"
        assertEquals("张三", student.getName("aa",1));
		//调用student.getName 第一个参数随便传入，第二个参数要传1。断言为 "李四"
        assertEquals("李四", student.getName("bb",1));
	}
```


# 六、when..thenThrow 模拟异常抛出
使用 when(mock.someMethod()).thenThrow(Exception()) 模拟方法异常抛出

```java
	@Mock
    Student student;

    @Test
    public void exceptionCorrect()  {
	    //模拟当调用 student.getName 时抛出 RuntimeException 异常
        when(student.getName(anyString(),anyInt())).thenThrow(new RuntimeException());
        //将抛出 RuntimeException 异常
        student.getName("aa",1);
	}
```


# 七、verify 验证方法是否被调用

有些时候，测试并不关心返回结果，而是关心方法是否被正确的参数调用过，这时候就应该使用验证方法了。

verify 用于验证模拟的实例方法是否被调用，若没有调用则验证失败，就报错提示：
```java
    @Mock
    Student student;

    @Test
    public void verifyCorrect()  {
        when(student.getName(anyString(),anyInt())).thenReturn("张三");
        assertEquals("张三", student.getName("aa",1));
		// 验证模拟的 student 实例其 getName 方法是否被调用过
		verify(student).getName(anyString(),anyInt());
	}
```


verify 还可以使用 times 来验证方法调用的次数，若实际调用次数和预期的不符合，就报错提示：
```java
    @Mock
    Student student;

    @Test
    public void verifyCorrect()  {
       when(student.getName(anyString(),eq(1))).thenReturn("张三","李四");
        assertEquals("张三", student.getName("aa",1));
        assertEquals("李四", student.getName("bb",1));
        //验证student.getName("aa",1)调用了1次
        verify(student,times(1)).getName("aa",1);
        //验证student.getName("bb",1)调用了1次
        verify(student,times(1)).getName("bb",1);
        //验证student.getName 总的调用了2次
        verify(student,times(2)).getName(anyString(),eq(1));
	}
```


# 八、Spy 运行真实方法

有些时候我们不想对一个对象进行 mock，但是我们想判断一个普通对象的方法有没有被调用过，那你可以使用 Spy 来监测对象，然后用 verify 来验证方法有没有被调用。

## （1）使用Spy方法
```java
    @Test
    public void spyCorrect() {
		//使用 spy 方法 监测 spyList
        List<String> spyList = spy(new ArrayList<>());
        spyList.add("one");
        spyList.add("two");

		//验证上面有没有调用add("one")方法
        verify(spyList).add("one");
        assertEquals(2, spyList.size());
        assertEquals("one", spyList.get(0));
        assertEquals("two", spyList.get(1));

        //size()和get(0)方法被模拟了返回值就不会去执行其真实方法，get(1)没被模拟会调用其真实方法返回值
        when(spyList.size()).thenReturn(100);
        when(spyList.get(0)).thenReturn("aa");
        assertEquals(100, spyList.size());
        assertEquals("aa", spyList.get(0));
        assertEquals("two", spyList.get(1));
	}
```

## （2）使用 @Spy 注解
除了上面使用 Spy 方法，你也可以使用 @Spy 注解达到一样的效果：
```java
    @Spy
    List<String> spyList = new ArrayList<>();
	
    @Test
    public void spyCorrect() {
        spyList.add("one");
        spyList.add("two");

		//验证上面有没有调用add("one")方法
        verify(spyList).add("one");
        assertEquals(2, spyList.size());
        assertEquals("one", spyList.get(0));
        assertEquals("two", spyList.get(1));

        //size()和get(0)方法被模拟了返回值就不会去执行其真实方法，get(1)没被模拟会调用其真实方法返回值
        when(spyList.size()).thenReturn(100);
        when(spyList.get(0)).thenReturn("aa");
        assertEquals(100, spyList.size());
        assertEquals("aa", spyList.get(0));
        assertEquals("two", spyList.get(1));
	}
```


# 九、@InjectMocks 注解解决依赖
上面第四点提到，初始化 mock 后，直接调用mock实例对象的方法不会走真实的方法，只会返回默认值。

但是有些时候我们不想直接 mock 模拟对象，我们想实际的运行对象的方法又让它返回一个模拟值，但是这个对象的方法里又依赖了其他的对象。这个时候就可以使用 @InjectMocks 注解了。


**@InjectMocks 创建一个类的实例，并将使用 @Mock 注解创建的 mock 注入到这个实例中。**

假设有 DatabaseDAO、NetworkDAO、MainClass 三个类，其中 MainClass 类中的 save 方法需要用到 DatabaseDAO、NetworkDAO 。

```java
  public class DatabaseDAO {
    public String save(String fileName) {
        System.out.println("Saved in database");
        return "Saved in database Ok";
    }
} 
```

```java
public class NetworkDAO {
    public String save(String fileName) {
        System.out.println("Saved in network location");
        return "Saved in network Ok";
    }
} 
```

```java
public class MainClass {
    DatabaseDAO database;
    NetworkDAO network;

    public boolean save(String fileName) {
        String databaseResule = database.save(fileName);
        System.out.println("Saved in database in Main class, "+databaseResule);

        String netWorkResule =  network.save(fileName);
        System.out.println("Saved in network in Main class, "+netWorkResule);

        return false;
    }
}
```

单元测试：
```java
@ExtendWith(MockitoExtension.class)
public class MainClassTest {

    @InjectMocks
	@Spy //加上@Spy 注解防止mock多线程运行报错
    MainClass mainClass;

    @Mock
    DatabaseDAO dependentClassOne;

    @Mock
    NetworkDAO dependentClassTwo;

    @Test
    public void injectMocksTest1() {
		//不使用when..thenReturn模拟返回值，调用方法执行后将返回真实返回值
        assertFalse(mainClass.save("temp.txt"));
        verify(mainClass).save("temp.txt");
    }

    @Test
    public void injectMocksTest2() {
        when(mainClass.save("temp.txt")).thenReturn(true);
		//使用when..thenReturn模拟返回值，调用方法执行后将返回模拟返回值
        assertTrue(mainClass.save("temp.txt"));
        verify(mainClass).save("temp.txt");
    }
}
```

运行此单元测试，输出结果：
```java
Saved in database in Main class, null
Saved in network in Main class, null
```

上面没对 dependentClassOne 和 dependentClassTwo 的 save 方法进行返回值模拟，所以默认返回了 null , 下面对他们模拟下返回值：
```java
@ExtendWith(MockitoExtension.class)
public class MainClassTest {

    @InjectMocks
	@Spy //加上@Spy 注解防止mock多线程运行报错
    MainClass mainClass;

    @Mock
    DatabaseDAO dependentClassOne;

    @Mock
    NetworkDAO dependentClassTwo;

    @Test
    public void injectMocksTest2() {
        when(dependentClassOne.save(anyString())).thenReturn("aa");
        when(dependentClassTwo.save(anyString())).thenReturn("bb");
        when(mainClass.save("temp.txt")).thenReturn(true);
        assertTrue(mainClass.save("temp.txt"));
        verify(mainClass).save("temp.txt");
    }
}
```


运行此单元测试，输出结果：
```java
Saved in database in Main class, aa
Saved in network in Main class, bb
```



参考：
[Mockito Tutorial](https://www.baeldung.com/mockito-series)