[TOC]

# 一、Optional对象的创建
1.使用of创建
```java
		Student student = new Student("王五", 80);
        Optional<Student> optional = Optional.of(student);
```

2.使用ofNullable创建
```java
		Student student = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student);
```
3.of和ofNullable的区别
of接收的值不能为null,否则会报空指针异常：
```java
		Student student = null;
        Optional<Student> optional = Optional.of(student);
```
ofNullable接收的值可以是null，不会报空指针异常：
```java
		Student student = null;
        Optional<Student> optional = Optional.ofNullable(student);
```



# 二、 isPresent()和isEmpty()判空处理
**1.isPresent():**
使用isPresent方法来判断非空，isPresent相当于!=null
```java
		
		Student student1 = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student1);
		//将输出：student1不为空的操作
        if (optional.isPresent()){
            System.out.println("student1不为空的操作");
        }else {
            System.out.println("student1为空的操作");
        }
		
		
		Student student2 = null;
        Optional<Student> optional2 = Optional.ofNullable(student2);
		//将输出：student2为空的操作
        if (optional2.isPresent()){
            System.out.println("student2不为空的操作");
        }else {
            System.out.println("student2为空的操作");
        }
```
**2.isEmpty():**
从Java 11开始，我们可以使用isEmpty方法来处理相反的操作 isEmpty相当于 ==null
```java
		
		Student student1 = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student1);
        //将输出：student1不为空的操作
        if (optional.isEmpty()){
            System.out.println("student1为空的操作");
        }else {
            System.out.println("student1不为空的操作");
        }


        Student student2 = null;
        Optional<Student> optional2 = Optional.ofNullable(student2);
        //将输出：student2为空的操作
        if (optional2.isEmpty()){
            System.out.println("student2为空的操作");
        }else {
            System.out.println("student2不为空的操作");
        }
```


# 三、 ifPresent()和ifPresentOrElse()的条件动作
**1.ifPresent:**
ifPresent相当于相判断 !=null,不为空才执行里面的代码:
```java
		Student student1 = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student1);
        optional.ifPresent(name -> System.out.println("student1不为空，姓名为："+name.getName()));
```
上面的代码和下面不使用Optional的代码效果是等效的：
```java
		Student student1 = new Student("王五", 80);
        if (student1 != null){
            System.out.println("student1不为空，姓名为："+student1.getName());
        }
```
**2.ifPresentOrElse():**
java 9里新增了ifPresentOrElse()，当Optional里的值不为空则执行第一个参数里的代码，为空则执行第二个参数的代码，相当于if-else :
```java
        Student student = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student);
        optional.ifPresentOrElse( value -> System.out.println("student不为空，姓名："+value.getName()), () -> System.out.println("student为空") );
```

# 四、 Optional对象中获取值
使用get来获取值：
```java
		Student student1 = new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student1);
        Student myStudent = optional.get();
        System.out.println("姓名："+myStudent.getName());//输出姓名：王五
```
需要注意的是如果student1为null,在使用get()获取值的时候会抛出NoSuchElementException异常：
（下面的写法只是样例，并不可取，后面会有几种代替isPresent()和get()）
```java
		Student student1 = null;
        Optional<Student> optional = Optional.ofNullable(student1);
        Student myStudent = optional.get();
        System.out.println("姓名："+myStudent.getName());
```

```java
		Student student1 = null;
        Optional<Student> optional = Optional.ofNullable(student1);
        if (optional.isPresent()){
            Student myStudent = optional.get();
            System.out.println("姓名："+myStudent.getName());
        }else {
            System.out.println("student1为空");
        }
```


# 五、orElse()与orElseGet()操作
orElse与orElseGet可替代get操作并且可以避免get操作出现的NoSuchElementException异常

**1.平常普通的代码：**
```java
		Student student1 = new Student("王五", 80);
        Student student2 = new Student("张三", 90);
        Student student3 = null;
        if (student1 != null) {
            student3 = student1;
        } else {
            student3 = student2;
        }
```
**2.使用orElse：**
```java
		Student student1 = new Student("王五", 80);
        Student student2 = new Student("张三", 90);
		Optional<Student> optional = Optional.ofNullable(student1);
		//先执行orElse里的语句，然后判断student1不为空则返回student1给 student3，否则返回student2给student3
        Student student3 = optional.orElse(student2);
```
**3.使用orElseGet：**
```java
		Student student1 = new Student("王五", 80);
        Student student2 = new Student("张三", 90);
		Optional<Student> optional = Optional.ofNullable(student1);
		//判断student1不为空则返回student1给 student3，否则返回student2给student3
        Student student3 = optional.orElseGet(()->student2);
```
**4.orElse与orElseGet区别：**
- 无论Optional中的值是否为空orElse中的代码都会执行，而orElseGet中的代码只有Optional中的值为空才会执行。
- orElseGet接收的是Supplier<T>类型的参数，而orElse接受的是T类型的参数；

当student1为空时orElse和orElseGet都会执行：
```java
	public static void main(String[] args) throws Exception {
        Student student1 = null;
        Optional<Student> optional = Optional.ofNullable(student1);

        Student student2 = optional.orElse(getMyStudent());
        Student student3 = optional.orElseGet(() -> getMyStudent());
	}
		
	public static Student getMyStudent() {
        System.out.println("开始执行getMyStudent");
        return new Student("张三", 90);
    }
```
当student1不为空时只有orElse会执行：
```java
	public static void main(String[] args) throws Exception {
        Student student1 =  new Student("王五", 80);
        Optional<Student> optional = Optional.ofNullable(student1);

        Student student2 = optional.orElse(getMyStudent());
        Student student3 = optional.orElseGet(() -> getMyStudent());
	}
		
	public static Student getMyStudent() {
        System.out.println("开始执行getMyStudent");
        return new Student("张三", 90);
    }
```

# 六、or()操作
Java 9引入了or()方法，or操作与orElse和orElseGet操作相似，只不过or操作返回的是一个新的Optional对象，而orElse和orElseGet操作返回的是Optional中的值
```java
        String expected = null;
        Optional<String> value = Optional.ofNullable(expected);
        Optional<String> defaultValue = Optional.of("default");
        //or()方法将返回一个新的Optional对象
        Optional<String> result = value.or(() -> defaultValue);
        System.out.println(result.get());//default
```


# 七、 filter()过滤操作

filter将谓词作为参数并返回一个Optional对象
```java
		Integer year = 2022;
        Optional<Integer> yearOptional = Optional.ofNullable(year);
        boolean is2022 = yearOptional.filter(y -> y == 2016).isPresent();
        System.out.println(is2022);//true
        boolean is2023 = yearOptional.filter(y -> y == 2017).isPresent();
        System.out.println(is2023);//false
```

# 八、 map()转换值操作
**1.map()：**
map将谓词作为参数并返回一个Optional对象

```java
        List<String> companyNames = Arrays.asList("paypal", "oracle", "", "microsoft", "", "apple");
        Optional<List<String>> listOptional = Optional.of(companyNames);
        //获取companyNames的长度
        int size = listOptional.map(List::size).orElse(0);
        System.out.println(size);//6
```

```java
        String password = " password ";
        Optional<String> passOpt = Optional.of(password);
        boolean correctPassword = passOpt.map(String::trim).filter(pass -> pass.equals("password")).isPresent();
        System.out.println(correctPassword);//true 
```
普通代码：
```java
        Student student1 = new Student("aaa", 80);
        String stuName=null;
        if (student1!=null) {
            String name = student1.getName();
            if (name!=null) {
                stuName=name.toUpperCase();
            }
        }
        System.out.println(stuName);//AAA
```
使用map()代替上面的普通代码：
```java
        Student student1 = new Student("aaa", 80);
        Optional<Student> optional = Optional.ofNullable(student1);
        String stuName=optional.map(u -> u.getName())
                .map(name -> name.toUpperCase())
                .orElse(null);
        System.out.println(stuName);
```


**2.filter和map的联合使用：**
Modem类
```java
public class Modem {
    private Double price;

    public Modem(Double price) {
        this.price = price;
    }
    // 省略 getters 和 setters
}
```
普通的判断价格是否在在10到15之间的写法：
```java
public boolean priceIsInRange1(Modem modem) {
    boolean isInRange = false;
    if (modem != null && modem.getPrice() != null && (modem.getPrice() >= 10 && modem.getPrice() <= 15)) {
        isInRange = true;
    }
    return isInRange;
}
```
而采用filter和map可以这么写：
```java
public boolean priceIsInRange1(Modem modem) {
    return Optional.ofNullable(modem2)
       .map(Modem::getPrice)
       .filter(p -> p >= 10)
       .filter(p -> p <= 15)
       .isPresent();
}
```
# 九、flatMap()解包操作
flatMap()方法作为转换值的替代方法。不同之处在于map仅在它们被解包时才转换值，而flatMap接受一个被包装的值并在转换之前解包它。
Person 类：
```java
public class Person {
    private String name;
    private int age;
    private String password;

    public Optional<String> getName() {
        return Optional.ofNullable(name);
    }

    public Optional<Integer> getAge() {
        return Optional.ofNullable(age);
    }

    public Optional<String> getPassword() {
        return Optional.ofNullable(password);
    }

    // 省略 getters 和 setters
}
```

```java
    Person person = new Person("john", 26);
    Optional<Person> personOptional = Optional.of(person);

    Optional<Optional<String>> nameOptionalWrapper  =  personOptional.map(Person::getName);
    Optional<String> nameOptional  = nameOptionalWrapper.orElseThrow(IllegalArgumentException::new);
    String name1 = nameOptional.orElse("");
    System.out.println("john".equals(name1));

    String name = personOptional
      .flatMap(Person::getName)
      .orElse("");
	  System.out.println("john".equals(name));
}
```
# 十、stream ()方法
在 Java 9中添加到Optional类的最后一个方法是stream()方法，允许我们将Optional实例视为Stream
```java
        Optional<List<String>> value = Optional.of(Arrays.asList("aaaa","bbbb","ccc"));
        List<String> collect = value.stream()
			.flatMap(Collection::stream)
			.filter(v->v.length()==4)
			.map(String::toUpperCase)
			.collect(Collectors.toList());
        System.out.println(collect);//[AAAA, BBBB]
}
```
---
参考：
[Guide To Java 8 Optional](https://www.baeldung.com/java-optional)
[Java | Optional的12种实践建议](http://t.zoukankan.com/jj81-p-14052547.html)
[java1.8的Optional和Stream流的简单运用](https://www.codeleading.com/article/13662375238/)