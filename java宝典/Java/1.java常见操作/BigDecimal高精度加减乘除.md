有时候我们计算金钱或者其他一些计算的时候需要高精度的计算加减乘除，可以使用BigDecimal

# 加

```java
BigDecimal num1 = new BigDecimal("100.569");
BigDecimal num2 = new BigDecimal("50.799");
BigDecimal data = num1.add(num2);
String result = String.valueOf(data);
System.out.println(result);//输出151.368
```

# 减

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.subtract(num2);
 String result = String.valueOf(data);
 System.out.println(result);//输出49.770
```

# 乘

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.multiply(num2);
 String result = String.valueOf(data);
 System.out.println(result);//输出5108.804631
```

# 除

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.divide(num2,5, RoundingMode.HALF_UP);//商保留5位小数
 String result = String.valueOf(data);
 System.out.println(result);//输出1.97974
```
# 四舍五入保留小数位数

```java
BigDecimal number = new BigDecimal("11.02156");
//四舍五入保留小数点后3位数
BigDecimal data = number.setScale(3, RoundingMode.HALF_UP);
String result = String.valueOf(data);
System.out.println(result);//输出11.022
```
# 绝对值
```java
BigDecimal num1 = new BigDecimal("-11.33");
BigDecimal num1Abs =  num1.abs();
System.out.println(num1Abs);//11.33
```
# 最大值与最小值
```java
BigDecimal num1 = new BigDecimal("33.33");
BigDecimal num2 = new BigDecimal("22.33");
//获取 num1 和 num2 中的最大值
BigDecimal max =  num1.max(num2);
System.out.println(max);//33.33
```
```java
BigDecimal num1 = new BigDecimal("33.33");
BigDecimal num2 = new BigDecimal("22.33");
//获取 num1 和 num2 中的最小值
BigDecimal min =  num1.min(num2);
System.out.println(min);//22.33
```
# 比较大小
使用 compareTo 可以比较大小，-1 则 num1 小于 num2，0 则等于，1 则大于
```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 if (num1.compareTo(num2 ) < 0) {
     System.out.println("num1小于num2");
 }
 if (num1.compareTo(num2 ) == 0) {
     System.out.println("num1等于num2");
 }
 if (num1.compareTo(num2 ) > 0) {
     System.out.println("num1大于num2");
 }
```



# 初始化

BigDecimal提供了0、1、10 这三个数的默认值创建：

```java
BigDecimal num1 =  BigDecimal.ZERO;
BigDecimal num2 =  BigDecimal.ONE;
BigDecimal num2 =  BigDecimal.TEN;
```
下面的代码和上面的代码是等价的：

```java
 BigDecimal num1 = new BigDecimal("0");
 BigDecimal num2 = new BigDecimal("1");
 BigDecimal num2 = new BigDecimal("10");
```

---
参考：
[DecimalFormat的使用](https://www.jianshu.com/p/b3699d73142e)
 [DecimalFormat除法](https://blog.csdn.net/lopper/article/details/5314686?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)