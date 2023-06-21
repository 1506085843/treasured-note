[TOC]

# 前言
本文为 JUnit 5单元测试（二）—— 断言
# 一、单元测试规范和操作

**1.单元测试的类名应该起为 xxxxTest.java 表明这个一个测试类，类名应该用简洁的英文表明测试内容或函数。**

- 例如：为了测试一个计算和的方法，可以取名为 SumTest.java

**2.测试方法上应该加上 @Test  注解以表明其是一个测试方法。**

- 运行测试方法：在 idea 中每个测试方法前都有一个运行按钮，点击后会运行该测试方法。

- 运行测试类：在 idea 中在测试类的类名前面有一个运行按钮，点击后会运行当前测试类中所有的测试方法。
-![请添加图片描述](https://img-blog.csdnimg.cn/b9c80efa56ab4cf8b44dd549e6fe8c13.jpeg) 你也可以查看单元测试的覆盖率![在这里插入图片描述](https://img-blog.csdnimg.cn/c896ff2c18b44f1ba1359f7acaa2f383.png)
 可以看到总的类覆盖率、方法覆盖率和行覆盖率
![在这里插入图片描述](https://img-blog.csdnimg.cn/071dfea539734f99b7e0b52c876c184b.png)
可以双击包名点进去查看具体不同的包的覆盖率
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e6d1ac921534369b9cb064b64d68916.png)


**3.使用 surefire 运行所有单元测试并生成报告。**

- 你可能会有很多单元测试类放在了 test/java/  文件夹下，你可以在 idea 中右侧maven页签中的 Lifecycle 下双击 test ，将会帮你运行所有的单元测试，每个单元测试类都会在 target/surefire-reports 下生成 txt 报告，里面有当前单元测试类的执行成功、失败、跳过、耗时等信息。

- 生成  txt  报告后，如果你不想一个个去看每个测试类生成的 txt 报告，可以在 idea 的 Terminal 命令行窗口输入` mvn surefire-report:report` 命令执行，它会读取所有 txt 报告在 target/site/  下生成 surefire-reports.html 文件，用浏览器打开该文件你就可以直观的看到所有单元测试类的所有执行情况。
- ![请添加图片描述](https://img-blog.csdnimg.cn/7a3e0e8640f341e99b6838ec49ac8f34.jpeg)


# 二、JUnit5提供的注解


| 注解           | 描述                                                     |
| -------------- | -------------------------------------------------------- |
| `@BeforeEach`  | 在方法上注解，在每个测试方法运行之前执行。               |
| `@AfterEach`   | 在方法上注解，在每个测试方法运行之后执行                 |
| `@BeforeAll`   | 该注解方法会在所有测试方法之前运行，该方法必须是静态的。 |
| `@AfterAll`    | 该注解方法会在所有测试方法之后运行，该方法必须是静态的。 |
| `@Test`        | 用于将方法标记为测试方法                                 |
| `@DisplayName` | 用于为测试类或测试方法提供说明。                         |
| `@Nested`      | 用于创建嵌套测试类。（注解在测试类的内部类上）           |
| `@Disable`     | 用于禁用或忽略测试类或测试方法                           |
| `@Tag`         | 用于标记测试方法或测试类                                 |
| `@TestFactory` | 标记一种方法是动态测试的测试工场                         |

说明：

- `@BeforeAll`：使用 @BeforeAll 注解的方法必须是用 static 修饰的，无论是运行某一个测试方法还是运行一个测试类，@BeforeAll 注解的方法都会在所有测试方法执行前运行，并只运行一次。
- `@BeforeEach`：每个测试方法运行之前执行。如果你运行一个测试类，每个测试方法执行前都会先执行一次@BeforeEach注解的方法。
- `@AfterEach`:每个测试方法运行之后执行。如果你运行一个测试类里，每个测试方法执行完后都会执行一次@AfterEach注解的方法。
- `@AfterAll`：使用 @AfterAll 注解的方法必须是用 static 修饰的，无论是运行某一个测试方法还是运行一个测试类，@AfterAll 注解的方法都会在所有测试方法执行完后运行，并只运行一次。
- `@Disable`:当运行一个测试类时被 @Disable 注解的测试方法会被忽略，不会被运行。

# 三、断言

## 1.什么是断言

断言就是专门用来验证输出和期望是否一致的一个工具。换句话说断言就是判断一个方法所产生的结果是否符合你期望的那个结果。

例如：如果你写个一个方法 sum( ) 用来计算两个数的和，那现在你想验证下你这个方法计算 1+1 是不是 2，你可以使用 assertEquals 断言方法，就像这样 assertEquals(2, sum(1,1)) ，如果 你 sum 方法计算 1+1 后返回的不是 2，那么测试方法断言就失败了这里就会报错提示你；如果返回的就是预期的2，那么就会继续执行 assertEquals 断言后的语句。

## 2.断言方法

除了上面提到的 assertEquals 断言方法，junt5 还提供了许多其他断言方法。

| 断言方法                                                     | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| assertEquals(expected, actual, message)                      | 如果预期值 expected 不等于实际返回值 actual ，则断言失败。   |
| assertNotEquals(expected, actual, message)                   | 如果预期值 expected 等于实际返回值 actual ，则断言失败。     |
| assertTrue(booleanExpression, message)                       | 如果 booleanExpression 不是 true ，则断言失败。              |
| assertFalse(booleanExpression, message)                      | 如果 booleanExpression 不是 false ，则断言失败。             |
| assertNull(actual, message)                                  | 如果 actual 不是 null ，则断言失败。                         |
| assertNotNull(actual, message)                               | 如果 actual 是 null ，则断言失败。                           |
| assertSame(object1, object2, message)                        | 如果两个对象引用没有指向同一个对象，则断言失败               |
| assertNotSame(object1, object2, message)                     | 如果两个对象引用指向同一个对象，则断言失败                   |
| assertArrayEquals(object1, object2, message)                 | 如果方两个数组不相等，则断言失败                             |
| assertIterableEquals(Iterable1,Iterable2, message)           | 如果方两个列表集合的元素顺序或大小不相等，则断言失败         |
| assertTimeout(expectedTimeout, executable, message)          | 如果执行的方法超过预期的时间，则断言失败（没执行完的代码会继续执行） |
| assertTimeoutPreemptively(expectedTimeout, executable, message) | 如果执行的方法超过预期的时间，则断言失败（没执行完的代码会结束执行） |
| fail(failmessage)                                            | 抛出异常                                                     |

（1）message ：上面这些断言方法里的 message 参数是可选的，当有 message  参数时如果断言失败时就会输出 message 内容；若没有 message  参数，断言失败则默认提示org.opentest4j.AssertionFailedError，例如：

```java
//如果断言失败会报错org.opentest4j.AssertionFailedError
assertEquals(3, sum(1,1));
//如果断言失败会报错org.opentest4j.AssertionFailedError: 断言失败啦！sum(1,1)得到的结果不是预期的2，请检查sum方法！
assertEquals(2, sum(1,1),"断言失败啦！sum(1,1)得到的结果不是预期的2，请检查sum方法！");
```

（2）fail 断言方法：上面的最后一个 fail 断言方法用来抛出异常，使测试方法运行到 fail 方法时立即失败终止（如果是运行的整个测试类，测试类里的其他测试方法将会继续运行，只会终止当前测试方法）。fail 方法一般用于当代码执行到了某个不应该被到达的分支时使用  fail 方法终止测试方法并给出提示信息；或者在catch块中抛出异常信息。示例：

```java
    @Test
    public void demo() {
        if ("a".equals("b")) {
            System.out.println("a等于b");
        }else {
            fail("a不等于b,抛出错误");
        }
    }

    @Test
    public void divisionCheck() {
        try {
            int a=2/0;
        } catch (Exception e) {
            fail(e.getMessage());
        }
    }
```
（3）assertIterableEquals 
- assertIterableEquals：用于比较两个Iterable对象（如List、Set等）中的元素是否相等（元素在集合对象中的顺序要一致），不需要这两个对象具有相同的类型。
```java
        List<String> arrList = new ArrayList<>(asList("Java", "Junit", "Test"));
        List<String> linList = new LinkedList<>(asList("Java", "Junit", "Test"));
		//arrList和linList虽然一个是ArrayList一个是LinkedList类型，但他们元素都相同，所以断言成功
        assertIterableEquals(arrList, linList);
```


（4）上面所有断言方法示例：

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.time.Duration;
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

import static java.util.Arrays.asList;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("断言测试类")
public class AssertTest {

    @DisplayName("断言测试方法")
    @Test
    public void testAssertions() {
        String str1 = new String("abc");
        String str2 = new String("abc");
        String str3 = null;
        String str4 = "abc";
        String str5 = "abc";
        int val1 = 5;
        int val2 = 6;
        String[] expectedArray = {"one", "two", "three"};
        String[] resultArray = {"one", "two", "three"};
        List<String> arrList = new ArrayList<>(asList("Java", "Junit", "Test"));
        List<String> linList = new LinkedList<>(asList("Java", "Junit", "Test"));

        assertEquals(str1, str2);
        assertNotEquals(3, 2, "error");
        assertTrue(val1 < val2);
        assertFalse(val1 > val2);
        assertNotNull(str1);
        assertNull(str3);
        assertSame(str4, str5, "断言失败，str4和str5指向的对象不一样");
        assertNotSame(str1, str3);
        assertArrayEquals(expectedArray, resultArray);
        assertIterableEquals(arrList, linList);
        assertTimeout(
                Duration.ofSeconds(2),
                () -> {
                    Thread.sleep(1000);
                }
        );
    }
}
```

所有断言成功，运行效果如下：

![请添加图片描述](https://img-blog.csdnimg.cn/5586e0c9f613423596d1633679b74334.jpeg)


# 四、assertThat断言
虽然上面的一些断言方法已经满足了日常的一些测试，但是还不够丰富和灵活。
例如：如果你一个字符串是只有等于 "abcdef",并且长度是6，以ab开始，并且包含 de 才算断言成功，那你需要写好几个断言语句或者判断方法，而 assertThat 就可以很方便的做到这一点。

assertThat 断言是第三方库 AssertJ-core 所提供的方法，AssertJ-core 提供了一组类和实用方法，使我们能够轻松地编写流畅而漂亮的断言。


**在 pom.xml 中添加 AssertJ-core 依赖：**

```xml
 <!-- assertj-core用于单元测试中使用assertThat断言-->
    <dependency>
      <groupId>org.assertj</groupId>
      <artifactId>assertj-core</artifactId>
      <version>3.24.2</version>
      <scope>test</scope>
    </dependency>
```
assertThat断言挺简单的，下面会用一些实例来快速介绍使用 assertThat 对常见类型的断言使用。
## 1. int断言
```java
    @Test
    public void intAssertThat() {
        int a = 22;
        //只有a为大于10小于30并且等于22时，断言才成功
        assertThat(a)
                .isGreaterThan(10)
                .isLessThan(30)
                .isEqualTo(22);
    }
```

## 2.字符断言
```java
    @Test
    public void intAssertThat() {
        int a = 22;
        //只有a为大于10小于30并且等于22时，断言才成功
        assertThat(a)
                .isGreaterThan(10)
                .isLessThan(30)
                .isEqualTo(22);
    }
```

## 3.字符串断言
```java
    @Test
    public void charAssertThat() {
        //只有c不等于a，且在 Unicode 表中，且大于w(即在w之后)，且是小写时断言才成功
        char c = 'x';
        assertThat(c)
                .isNotEqualTo('a')
                .inUnicode()
                .isGreaterThanOrEqualTo('w')
                .isLowerCase();
    }
```

## 4.boolean断言
```java
    @Test
    public void boolAssertThat() {
        boolean bo1 = true;
        boolean bo2 = true;
        //只有bo1为true并且等于bo2时断言才成功
        assertThat(bo1)
                .isTrue()
                .isEqualTo(bo2);
    }
```

## 5.List和数组断言
5.1 对 List 和数组断言：
```java
    @Test
    public void listArrayAssertThat1() {
        //只有list1不为空（不为null且size大于0）且包含元素1，且包含元素3和4，且所有元素不为null，且等于list2 时，断言才成功
        List<String> list1 = Arrays.asList("1", "2", "3", "4");
        List<String> list2 = Arrays.asList("1", "2", "3", "4");
        assertThat(list1)
                .isNotEmpty()
                .contains("1")
                .containsSequence("3","4")
                .doesNotContainNull()
                .isEqualTo(list2);
		
		//只有array1不为null且包含元素3，且等于array2 ,且数组长度是3时，断言才成功		
        String[] array1 ={"1", "2", "3"};
        String[] array2 ={"1", "2", "3"};
        assertThat(array1)
                .isNotNull()
                .contains("3")
                .isEqualTo(array2)
                .hasSize(3);
    }
```
5.2 对 List 的每个元素过滤断言：
```java
    @Test
    public void listArrayAssertThat2() {
        List<String> list = Arrays.asList("1-a", "2-b", "3-c", "4-d","5-e ok");
        //只有list中每个元素都含有"-",并且至少有一个元素含有"ok"时，断言才成功
        assertThat(list)
                .allMatch(v->v.contains("-"))
                .anyMatch(v->v.contains("ok"));
    }
```
5.3 对List的进行过滤得到新的一个List进行断言：

假设有Student类如下
```java
public class Student {
    public String name;
    public int id;

    public Student(String name, int id) {
        this.name = name;
        this.id = id;
    }
	//省略name和id的get/set方法
	//...
}
```
```java
    @Test
    public void listArrayAssertThat2() {
       List<Student> stuList = new ArrayList<>();
        stuList.add(new Student("Jerry",1));
        stuList.add(new Student("Tom",2));
        stuList.add(new Student("Lucy",3));

        //过滤筛选出 stuList 里所有 name 叫 Tom 的，然后断言其数量只有一个
        assertThat(stuList)
                .filteredOn(v->"Tom".equals(v.getName()))
                .hasSize(1);
    }
```
再来一个例子：

```java
    @Test
    public void listFilterAssertThat() {
        List<Student> stuList = new ArrayList<>();
        Student student1 = new Student("张三",1);
        Student student2 = new Student("李四",2);
        Student student3 = new Student("王五",3);
        stuList.add(student1);
        stuList.add(student2);
        stuList.add(student3);

        List<String> nameList = Arrays.asList("张三","李四","王五");

        //extracting函数调用 getName 获取所有学生的姓名为一个新的List，然后只有这个新的List包含元素李四，且这个新是List等于nameList，断言才成功
        assertThat(stuList)
                .extracting(Student::getName)
                .contains("李四")
                .isEqualTo(nameList);
    }
}
```



## 6.Map断言

```java
    @Test
    public void mapAssertThat() {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(1, 5);
        map.put(2, 7);
        map.put(3, 15);
        //只有map不为空（不为null且size大于0），且包含key是2，且不包含key是10，且包含value是5，且包含键值分别是3和15时，断言才成功
        assertThat(map)
                .isNotEmpty()
                .containsKey(2)
                .doesNotContainKeys(10)
                .containsValue(5)
                .contains(entry(3, 15));
    }
```

## 7.对象断言
断言两个对象的所有字段的值相等
```java
    @Test
    public void objectAssertThat() {
		//假设你有Student类
        Student student1 = new Student("张三",1);
        Student student2 = new Student("张三",1);
        //只有student1的所有字段的值分别等于student2的字段的值时，断言才成功
        assertThat(student1)
                .usingRecursiveComparison()
                .isEqualTo(student2);
    }
```

对对象的字段进行断言：
```java
    @Test
    public void objectAssertThat() {
        //假设你有Student类
        Student student = new Student("tom jeff",4);
        //student的name字段的值包含jeff时，断言才成功
        assertThat(student.getName())
                .contains("jeff");

        //student的id字段的值大于3小于5时，断言才成功
        assertThat(student.getId())
                .isLessThan(5)
                .isGreaterThan(3);
    }
```

# 五、参数化测试
假如需要判断一个数字是不是奇数，你可能会写一个测试方法然后去断言判断，那现在如果你想测试一批其他数字呢？
这时候就可以用参数化测试，就相当于从那一批数据中每次取一个数据传入测试方法进行测试。

## 1.参数化测试的规则
(1) 测试方法上使用 @ParameterizedTest 注解代替 @Test 注解，以表明其是一个参数化测试方法；

(2) 你可以使用 @ValueSource 或 @EnumSource 或 MethodSource 或 @CsvFileSource ，来分别从值类型、枚举类型、方法类型、外部csv文件类型来传入所有你要测试的所有数据。下面就介绍下这几种注解的参数化测试使用方法。

## 2.值类型
值类型是使用 @ValueSource 来给参数化测试方法传递一批数据，@ValueSource中可接收的数据类型是：
- short (使用 shorts )
- byte (使用 bytes )
- int (使用 ints )
- long (使用 longs )
- float (使用 floats )
- double (使用 doubles )
- char (使用 chars )
- java.lang.String (使用 strings )
- java.lang.Class (使用 classes )

下面使用参数化测试来分别测试 1, 3, 5, -3, 15, Integer.MAX_VALUE 这几个数是不是奇数：
```java
    @ParameterizedTest
    @ValueSource(ints = {1, 3, 5, -3, 15, Integer.MAX_VALUE})
    void isOddNumbers(int number) {
        assertTrue(isOdd(number));
    }
    
    public static boolean isOdd(int number) {
        return number % 2 != 0;
    }
```
测试结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1eb420bf5d6c4ba4807fca6335297ea5.png)



## 2.枚举类型
使用@EnumSource可以从枚举类里传递数据给参数化测试方法进行测试


假设拥有Month枚举类如下：
```java
public enum Month {
    January(1), February(2), March(3), April(4), May(5), June(5), July(7), August(8), September(9), October(10), November(11), December(12);

    private final  int month;

    Month(int month){
        this.month = month;
    }

    public int getValue() {
        return month;
    }
}
```
单元测试方法：
```java
    @ParameterizedTest
    @EnumSource(Month.class) // 从Month枚举类里每次取一个月份
    void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
        int monthNumber = month.getValue();
        assertTrue(monthNumber >= 1 && monthNumber <= 12);
    }
```
测试结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a326da7c377d41769738d4a53a9329f4.png)


如果你不想测试Month类里的所有枚举。只想测试Month类里的三月和五月，可以使用names来过滤出它们进行测试

```java
    @ParameterizedTest
    @EnumSource(value = Month.class, names = {"March", "May"})
    void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
        int monthNumber = month.getValue();
        assertTrue(monthNumber >= 1 && monthNumber <= 12);
    }
```

## 3.外部csv文件类型
上面的值类型和枚举类型每次都只能传递一个值，而使用外部csv文件类型可以每次传递多个值。


**示例：**
在resources资源文件夹下新建students.csv文件，内容如下：
```java
id,姓名,分数
1,Tom,80
2,Jerry,90
3,Merry,75
```
下面的测试方法将读取students.csv文件的每一行，每次传入id,name,score这三个参数进行测试。
numLinesToSkip是跳过students.csv文件第1的标题（如果你的csv文件没有标题可去除numLinesToSkip属性）
```java
    @ParameterizedTest
    @CsvFileSource(resources = "/students.csv", numLinesToSkip = 1)
    void checkStudent(int id,String name, int score) {
        assertThat(id).isLessThan(10);				//断言id是否小于10
        assertThat(name).hasSizeGreaterThan(2);		//断言name长度是否大于2
        assertThat(score).isGreaterThan(60);		//断言score是否大于60
    }
```


## 4.方法类型
上面的三种方式参数来源都比较简单，并且都有局限性。使用它们很难传递复杂的对象，而使用方法类型可以提供更复杂的参数。

方法类型使用 @MethodSource 注解，其传递参数过来的方法**必须是 static 修饰的**静态方法，并且其返回值必须是stream流或数组（如:Stream, DoubleStream, LongStream, IntStream, Collection, Iterator, Iterable、Object[], String[]等）


**示例(1)：**
```java
    @ParameterizedTest
    @MethodSource("provideStringsForIsBlank")
    void isBlankOrNullStrings(String input, boolean expected) {
        boolean actual = input == null || input.trim().isEmpty();
        assertEquals(expected,actual);
    }

  //返回一组数据，该组数据第一个参数是字符串，第二个参数是表明第一个参数是否为空
    private static Stream<Arguments> provideStringsForIsBlank() {
        return Stream.of(
                Arguments.of(null, true),
                Arguments.of("", true),
                Arguments.of("  ", true),
                Arguments.of("abc", false)
        );
    }
```

**示例(2)：**
getStudents方法返回List<Student>流（相当于返回多个Student对象）测试
```java
    @ParameterizedTest
    @MethodSource("getStudents")
	//getStudents将返回List<Student>流，studentsCheck接收3次流里的数据测试
    void studentsCheck(Student student) {
        assertThat(student.getName()).hasSizeGreaterThan(2); //断言该学生的姓名长度大于2
        assertThat(student.getId()).isLessThan(5);  		//断言该学生的id小于5
    }


    private static Stream<Student> getStudents() {
        List<Student> stuList = new ArrayList<>();
        stuList.add(new Student("tom",1));
        stuList.add(new Student("jerry",2));
        stuList.add(new Student("lory",3));
        return stuList.stream();
    }
```

**示例(3)：**
接收方法返回的多个复杂参数测试
```java
    @ParameterizedTest
    @MethodSource("getArguments")
    //getArguments方法将传递两次数据过来测试，一次是(stuList1, intList1, "99")，一次是stuList2, intList2, "100")
    void studentsCheck(List<Student> stuList, List<Integer> intList, String score) {
        System.out.println(intList);
        System.out.println(score);
        //断言stuList中的每个学生姓名都包含"_"
        assertThat(stuList).allMatch(v -> v.getName().contains("_"));
        //断言stuList中有学生姓名叫"lory_3"
        assertThat(stuList).extracting(v -> v.getName()).contains("lory_3");
        //断言intList中的每个数都在0到7之间
        assertThat(intList).allMatch(v -> v > 0 && v < 7);
        //断言score字符串长度大于1
        assertThat(score).hasSizeGreaterThan(1);
    }

    private static Stream<Arguments> getArguments() {
        List<Student> stuList1 = new ArrayList<>();
        stuList1.add(new Student("tom_1", 1));
        stuList1.add(new Student("jerry_2", 2));
        stuList1.add(new Student("lory_3", 3));

        List<Student> stuList2 = new ArrayList<>();
        stuList2.add(new Student("tom_4", 4));
        stuList2.add(new Student("jerry_5", 5));
        stuList2.add(new Student("lory_3", 3));

        List<Integer> intList1 = Arrays.asList(1, 2, 3);
        List<Integer> intList2 = Arrays.asList(4, 5, 6);

        List<Arguments> arguments = new LinkedList<>();
        arguments.add(Arguments.of(stuList1, intList1, "99"));
        arguments.add(Arguments.of(stuList2, intList2, "100"));
        return arguments.stream();
    }
```


到这里你应该能够很熟练的使用单元测试和断言了，上面的断言示例只是列举了常用的一些断言方法，还有文件流断言、类断言等这里没做过多的解释，其他完整的方法和解释可参考下面的链接。


---

参考：
[AssertJ - fluent assertions java library](https://assertj.github.io/doc/)
[添加链接描述](https://assertj.github.io/doc/#assertj-overview)
[Introduction to AssertJ](https://www.baeldung.com/introduction-to-assertj)
[usingRecursiveComparison说明](https://javadoc.io/doc/org.assertj/assertj-core/3.13.2/org/assertj/core/api/AbstractObjectAssert.html#usingRecursiveComparison())
[JUNIT 5: USING LISTS AS AN ARGUMENT FOR PARAMETERIZED TESTS](https://blog.felix-seifert.com/junit-5-parameterized-tests-using-lists-as-argument/)
[Single Assert Call for Multiple Properties in Java Unit Testing](https://www.baeldung.com/java-testing-single-assert-multiple-properties)