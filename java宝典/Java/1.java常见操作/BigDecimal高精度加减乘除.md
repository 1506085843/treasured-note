有时候我们计算金钱或者其他一些计算的时候需要高精度的计算加减乘除，可以使用BigDecimal

**加：**

```java
BigDecimal num1 = new BigDecimal("100.569");
BigDecimal num2 = new BigDecimal("50.799");
BigDecimal data = num1.add(num2);
String result = String.valueOf(data);
System.out.println(result);//输出151.368
```

**减：**

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.subtract(num2);
 String result = String.valueOf(data);
 System.out.println(result);//输出49.770
```

**乘：**

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.multiply(num2);
 String result = String.valueOf(data);
 System.out.println(result);//输出5108.804631
```

**除：**

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 BigDecimal data = num1.divide(num2,5, RoundingMode.HALF_UP);//商保留5位小数
 String result = String.valueOf(data);
 System.out.println(result);//输出1.97974
```

**比较大小：**

```java
 BigDecimal num1 = new BigDecimal("100.569");
 BigDecimal num2 = new BigDecimal("50.799");
 int flag = num1 .compareTo(num2 )
```
flag = -1, 表示 num1 小于 num2 ；
flag = 0, 表示 num1 等于 num2 ；
flag = 1, 表示 num1 大于 num2 ；

---
参考：
[DecimalFormat的使用](https://www.jianshu.com/p/b3699d73142e)
 [DecimalFormat除法](https://blog.csdn.net/lopper/article/details/5314686?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)