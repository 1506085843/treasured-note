[TOC]


## 前言
jdk8之前日期时间相关的操作大多用的是Date类或者Calendar类。

比如：

```java
	Date date = new Date();
	SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	System.out.println(df.format(date)); //2020-09-22 09:19:44
```

|Date、Calendar缺点|说明|
|--------------------- | :------------------------------------------:|
| 非线程安全| java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java的日期类最大的问题之一。 |
| 设计很差| Java的日期时间类的定义并不一致，在 java.util 和 java.sql 的包中都有日期类，此外用于格式化和解析的类在 java.text 包中定义。java.util.Date 同时包含日期和时间，而 java.sql.Date 仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。 |
| 时区处理麻烦| 日期类并不提供国际化，没有时区支持，因此Java引入java.util.Calendar 和 java.util.TimeZone 类，但他们同样存在上述所有的问题。 |

jd8 以后增加了 **LocalDate、LocalTime、LocalDateTime** 和 **Zoned**，能更方便优雅的处理日期时间，本文主要介绍它们。

## 一、LocalDate、LocalTime、LocalDateTime介绍
| 日期时间类    | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| LocalDate     | 只包含年月日，不包含时分秒，当不需要具体的时间时可以使用 LocalDate |
| LocalTime     | 只包含时分秒，不包含年月日，当不需要具体的日期时可以使用 LocalTime |
| LocalDateTime | 包含年月日时分秒，当日期和时间都需要使用时可以使用 LocalDateTime |

## 二、获取当前时间

- LocalDate获取当前年、月、日

```java
        LocalDate nowdata = LocalDate.now();
        System.out.println(nowdata);   //2021-04-06 获取当前年月日
        System.out.println(nowdata.getYear());//2021 获取当前年份
        System.out.println(nowdata.getMonth().getValue());//4 获取当前月份
        System.out.println(nowdata.getDayOfMonth());//6 获取今天几号
        System.out.println(nowdata.getDayOfWeek().getValue());//2 获取今天星期几
        System.out.println(nowdata.getDayOfYear());//96 获取今天是今年的第几天
```

- LocalTime获取当前时、分、秒

```java
        LocalTime nowTime = LocalTime.now();
        System.out.println(nowTime);//09:51:11.987 获取当前的时间
        System.out.println(nowTime.getHour());//9 获取当前时间的小时
        System.out.println(nowTime.getMinute());//51 获取当前时间的分钟
        System.out.println(nowTime.getSecond());//11 获取当前时间的秒
```

- LocalDateTime获取当前年、月、日、时、分、秒
 （LocalDateTime相当于LocalDate + LocalTime的合体）


```java
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);//2021-04-06T09:51:11.987  获取当前的年月日时分秒
        System.out.println(localDateTime.getYear());//2021 获取当前年份
        System.out.println(localDateTime.getMonth().getValue());//4 获取当前月
        System.out.println(localDateTime.getDayOfMonth());//6 获取当前日
        System.out.println(localDateTime.getHour());//9 获取当前时
        System.out.println(localDateTime.getMinute());//51 获取当前分
        System.out.println(localDateTime.getSecond());//11 获取当前秒
        System.out.println(localDateTime.getDayOfYear());//96 今天是今年的第几天
        System.out.println(localDateTime.getDayOfWeek().getValue());//2 今天是星期几
```

## 三、日期和时间格式化

- LocalDate格式化

```java
		LocalDate getdata = LocalDate.now();
        DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy.MM.dd");
        System.out.println(getdata.format(f1));//2021.04.06
```

- LocalTime格式化

```java
        LocalTime getTime = LocalTime.now();
        DateTimeFormatter f1 = DateTimeFormatter.ofPattern("HH-mm-ss");
        System.out.println(getTime.format(f1));//16-27-08
```

- LocalDateTime 格式化

```java
        LocalDateTime localDateTime = LocalDateTime.now();
        DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        System.out.println(localDateTime.format(f1));//2021-04-06 10:04:32
```



```java
        LocalDateTime localDateTime = LocalDateTime.now();
        DateTimeFormatter f2 = DateTimeFormatter.ofPattern("yyyy.MM.dd-HH:mm");
        System.out.println(localDateTime.format(f2));//2021.04.06-10:04
```

```java
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime.toString());//2021-04-06T11:33:59.617
```

添加时区：
```java
        LocalDateTime localDateTime = LocalDateTime.now();
        OffsetDateTime date =localDateTime.atOffset(ZoneOffset.ofHours(+8));
        System.out.println(date.toString());//2021-04-06T10:55:09.599+08:00
```
注意：
LocalDateTime 当秒数刚好为0的时候格式化后秒会被省略，格式化要指定到秒。

```java
        LocalDateTime localDateTime = LocalDateTime.parse("2021-04-06T10:00:00");

        String time = localDateTime.toString();
        System.out.println(time);//2021-04-06T10:00

        DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String time1 = localDateTime.format(f);
        System.out.println(time1);//2021-04-06 10:00:00
```

## 四、字符串与LocalDate、LocalTime、LocalDateTime之间的互换
- 日期字符串转LocalDate类型

```java
        LocalDate getdata = LocalDate.parse("2021-04-06");//年月日用-分开，月份和日期如果小于10要补零
        System.out.println(getdata);//2021-04-06
```
- 20220805 转 2022-08-05：

```java
		String dayAfterTommorrow = "20220805";
		//除了DateTimeFormatter.BASIC_ISO_DATE格式DateTimeFormatter里还有其他格式可选择
        LocalDate formatted = LocalDate.parse(dayAfterTommorrow, DateTimeFormatter.BASIC_ISO_DATE);
        System.out.println("格式化后的日期为:  "+formatted);//格式化后的日期为:  2022-08-05
```

- 时间字符串转LocalTime类型

```java
        //时间字符串转LocalTime类型
        LocalTime gettime = LocalTime.parse("16:59:09");//时分秒用:分开，时分秒如果小于10要补零
        System.out.println(gettime);
```
- 日期时间字符串转LocalDateTime类型
（年月日之间要用-分割，时分秒用:分割，日期和时间之间用T分割）
```java
        LocalDateTime localDateTime = LocalDateTime.parse("2021-04-06T10:13:12");
        System.out.println(localDateTime);//2021-04-06T10:13:12
```

- LocalDate、LocalTime、LocalDateTime类型转为字符串直接toString

```java
        LocalDate nowdata = LocalDate.now();
        System.out.println(nowdata.toString());//2021-04-16

        LocalTime nowTime = LocalTime.now();
        System.out.println(nowTime.toString());//16:16:04.372

        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime.toString());//2021-04-16T16:16:04.373
```
## 五、时间创建（手动设置指定时间）

```java
        LocalDateTime ofTime = LocalDateTime.of(2022, 4, 21, 8, 8, 8);
        System.out.println("当前精确时间：" + ofTime);

        LocalDate localDate = LocalDate.of(2022, 4, 21);
        System.out.println("当前日期：" + localDate);

        LocalTime localTime = LocalTime.of(12, 1, 1);
        System.out.println("当天时间：" + localTime);
```
输出：

```java
当前精确时间：2022-04-21T08:08:08
当前日期：2022-04-21
当天时间：12:01:01
```

## 六、日期和时间的加减
日期的加减：

```java
		LocalDate today = LocalDate.now();
        System.out.println("今天的日期为:"+today);
        //plus() 方法用来增加天、周、月，ChronoUnit 类声明了这些时间单位
        //可以用同样的方法增加 1 个月、1 年、1 小时、1 分钟甚至一个世纪
        LocalDate nextWeek = today.plus(2, ChronoUnit.WEEKS);
        System.out.println("两周后的日期为:"+nextWeek);
```
**输出：**

```java
今天的日期为:2022-08-14
一周后的日期为:2022-08-28
```
日期时间的加减：
```java
        LocalDateTime now = LocalDateTime.now();
        System.out.println("当前时间："+now);
        LocalDateTime plusTime = now.plusMonths(1).plusDays(4).plusHours(1).plusMinutes(1).plusSeconds(1);
        LocalTime plusTime1 = now.plusHours(3).toLocalTime();
        System.out.println("增加1月4天1小时1分钟1秒时间后：" + plusTime);
        System.out.println("3个小时后的时间为:"+plusTime1);
        LocalDateTime minusTime = now.minusMonths(2).minusDays(3);
        System.out.println("减少2个月3天时间后：" + minusTime);
```
**输出：**

```java
当前时间：2022-04-21T10:30:06.155
增加1月4天1小时1分钟1秒时间后：2022-05-25T11:31:07.155
减少2个月3天时间后：2022-02-18T10:30:06.155
```

## 七、判断平年和闰年
- 指定具体年月日

```java
        LocalDate localDate = LocalDate.of(1999,1,7);//设置指定日期
        boolean bo = localDate.isLeapYear();//闰年为ture,平年是false
        System.out.println(bo);//false
```
- 指定年份

```java
	    boolean bo1 = IsoChronology.INSTANCE.isLeapYear(1999);//指定年份
		System.out.println(bo);//false
```
- LocalDateTime和LocalDate 判断平年闰年

```java
		LocalDateTime now = LocalDateTime.now();
        System.out.println("当前时间：" + now);
        System.out.println("今年是否闰年：" + Year.isLeap(now.getYear()));

        LocalDate data=LocalDate.now();
        System.out.println("当前日期：" + data);
        System.out.println("今年是否闰年：" + Year.isLeap(data.getYear()));
```
输出：

```java
当前时间：2022-04-21T10:51:57.825
今年是否闰年：false
当前日期：2022-04-21
今年是否闰年：false
```

## 八、判断指定日期是星期几

```java
		LocalDate localDate = LocalDate.of(2021, 1, 25);
        DayOfWeek dayOfWeek = localDate.getDayOfWeek();
        System.out.println(dayOfWeek);//MONDAY
```

```java
	    LocalDate localDate = LocalDate.of(2021,1,25);
	    //判断是不是星期一
        System.out.println(localDate.getDayOfWeek()== DayOfWeek.MONDAY);//true
```
判断今天是星期几：
```java
		DayOfWeek dayOfWeek = LocalDate.now().getDayOfWeek();
        System.out.println(dayOfWeek);
```


## 九、计算指定日期的月份有多少天
获取天数：
```java
	    LocalDate endDate = LocalDate.of(2021, 2, 1);
        int monthDay = endDate.lengthOfMonth();
        System.out.println(monthDay);//输出28，2021年2月有28天
```
获取本月最后一天的时间或日期：
```java
        LocalDateTime now = LocalDateTime.now();
        System.out.println("当前时间：" + now);
        LocalDateTime lastDay = now.with(TemporalAdjusters.lastDayOfMonth());
        System.out.println("本月最后一天的当前时间:" + lastDay);


        LocalDate nowData = LocalDate.now();
        System.out.println("当前日期：" + now);
        LocalDate lastDay1 = nowData.with(TemporalAdjusters.lastDayOfMonth());
        System.out.println("本月最后一天:" + lastDay1);
```

## 十、比较两个时间的早晚

```java
        LocalTime gettimeStart = LocalTime.parse("00:01:40");
        LocalTime gettimeEnd = LocalTime.parse("00:03:20");
	   //-1表示早于，1表示晚于，0则相等
        int value = gettimeStart.compareTo(gettimeEnd);
        System.out.println(value);
```

## 十一、计算两个时间相差多久（倒计时）

```java
        LocalTime gettimeStart = LocalTime.parse("00:01:40");
        LocalTime gettimeEnd = LocalTime.parse("00:03:20");
        System.out.println("相差: "+HOURS.between(gettimeStart, gettimeEnd)+"小时");//相差: 0小时
        System.out.println("相差: "+MINUTES.between(gettimeStart, gettimeEnd)+"分钟");//相差: 1分钟
        System.out.println("相差: "+SECONDS.between(gettimeStart, gettimeEnd)+"秒");//相差: 100秒
```

如果要计算出xx:xx:xx秒这样的结果：

```java
/**
     * 计算两个时间的时间差
     *
     * @param startime 开始时间，如：00:01:09
     * @param endtime  结束时间，如：00:08:27
     * @return  返回xx:xx:xx形式，如：00:07:18
     */
    public static String calculationEndTime(String startime, String endtime) {
        LocalTime timeStart = LocalTime.parse(startime);
        LocalTime timeEnd = LocalTime.parse(endtime);
        long hour = HOURS.between(timeStart, timeEnd);
        long minutes = MINUTES.between(timeStart, timeEnd);
        long seconds = SECONDS.between(timeStart, timeEnd);
        minutes = minutes > 59 ? minutes % 60 : minutes;
        String hourStr = hour < 10 ? "0" + hour : String.valueOf(hour);
        String minutesStr = minutes < 10 ? "0" + minutes : String.valueOf(minutes);
        long getSeconds = seconds - (hour * 60 + minutes) * 60;
        String secondsStr = getSeconds < 10 ? "0" + getSeconds : String.valueOf(getSeconds);
        return hourStr + ":" + minutesStr + ":" + secondsStr;
    }
```

## 十二、比较两个日期的早晚
比较两个日期的早晚：
```java
        LocalDate startDate = LocalDate.of(2020, 1, 18);
        LocalDate endDate = LocalDate.of(2021, 5, 17);
        //startDate是否早于endDate
        System.out.println(startDate.isBefore(endDate));//true
        //endDate是否晚于startDate
        System.out.println(endDate.isAfter(startDate));//true

        LocalDate startDate1 = LocalDate.of(2020, 1, 18);
        //startDate是否等于startDate1
        System.out.println(startDate.equals(startDate1));//true
```

或者使用Period.between：
```java
        LocalDate startDate = LocalDate.of(2020, 1, 18);
        LocalDate endDate = LocalDate.of(2021, 5, 17);
        Period period = Period.between(startDate, endDate);
        //判断startDate是不是早于endDate，早于则false，否则true
        System.out.println(period.isNegative());//false
```

比较两个日期是否相等：
```java
		LocalDate date1 = LocalDate.now();
        LocalDate date2 = LocalDate.of(2022,8,14);
        if(date1.equals(date2)){
            System.out.println("时间相等");
        }else{
            System.out.println("时间不等");
        }
```

## 十三、计算两个日期相隔多久

```java
LocalDate startDate = LocalDate.of(2020, 9, 27);
        LocalDate endDate = LocalDate.of(2030, 10, 2);
        System.out.println("总相差的天数:" + startDate.until(endDate, ChronoUnit.DAYS));//总相差的天数:3657
        System.out.println("总相差的月数:" + startDate.until(endDate, ChronoUnit.MONTHS));//总相差的月数:120
        System.out.println("总相差的年数:" + startDate.until(endDate, ChronoUnit.YEARS));//总相差的年数:10
        Period period = Period.between(startDate, endDate);
        System.out.println(
            "相差:" + period.getYears() + " 年 " +
                   period.getMonths() + " 个月 " +
                   period.getDays() + " 天");  //相差:10 年 0 个月 5 天
```





## 十四、比较两个日期时间的早晚
比较两个日期时间的早晚：
```java
        LocalDateTime date = LocalDateTime.parse("2019-03-03T12:30:30");
        LocalDateTime date1 = LocalDateTime.parse("2017-03-03T12:30:30");
        System.out.println(date.isAfter(date1));//true
        System.out.println(date.isBefore(date1));//false
        System.out.println(date.isEqual(date1));//false
```
两个日期时间是否相等：
```java
        LocalDateTime time1 = LocalDateTime.parse("2019-03-03T12:30:30");
        LocalDateTime time2 = LocalDateTime.parse("2019-03-03T12:40:30");
        if(time1.equals(time2)){
            System.out.println("时间相等");
        }else{
            System.out.println("时间不等");
        }
```

## 十五、计算某年某月有几个星期五

```java
public static void main(String[] args) {
        int a=numberOfDaysOfWeekInMonth(DayOfWeek.FRIDAY, YearMonth.of(2020, 9));
        System.out.println(a);
}

 public static int numberOfDaysOfWeekInMonth(DayOfWeek dow, YearMonth yearMonth) {
        LocalDate startOfMonth = yearMonth.atDay(1);
        LocalDate first = startOfMonth.with(TemporalAdjusters.firstInMonth(dow));
        LocalDate last = startOfMonth.with(TemporalAdjusters.lastInMonth(dow));
        return (last.getDayOfMonth() - first.getDayOfMonth()) / 7 + 1;
    }
```
说明：

**DayOfWeek.FRIDAY：** DayOfWeek枚举类中的星期五

**YearMonth.of**：此方法接收一个年份和月份，返回一个YearMonth类型

**yearMonth.atDay**：此方法接收一个指定的号数（范围1`到`366），返回LocalDate类型（即年月日）

**with**：方法返回此日期的调整副本。

**TemporalAdjusters.firstInMonth**：接收一个星期几参数，返回当前月份中第一个星期几的年月日

**TemporalAdjusters.lastInMonth**：接收一个星期几参数，返回当前月份中最后一个星期的年月日


## 十六、MonthDay及周期性判断
MonthDay表示月和日的组合

```java
        MonthDay mday = MonthDay.now();
        System.out.println(mday.getDayOfMonth());//获取今天是当月的几号,输出;14
        System.out.println(mday.getMonth());//获取今天是几月,输出AUGUST
        LocalDate date = mday.atYear(2021);//指定一个年份
        System.out.println(date);//输出：2021-08-14
```
生日判断：
（只要当天的日期和生日匹配，无论是哪一年都会打印出祝贺信息）
```java
		LocalDate date1 = LocalDate.now();
        LocalDate date2 = LocalDate.of(1999,8,14);
        MonthDay birthday = MonthDay.of(date2.getMonth(),date2.getDayOfMonth());//通过月份和日期指定MonthDay
        MonthDay currentMonthDay = MonthDay.from(date1);//LocalDate格式换转换为MonthDay
        if(currentMonthDay.equals(birthday)){
            System.out.println("是你的生日");
        }else{
            System.out.println("你的生日还没有到");
        }
```
## 十七、Clock时间戳与其他时区时间
Java 8 增加了一个 Clock 时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到System.currentTimeInMillis() 和 TimeZone.getDefault() 的地方都可用 Clock 替换。

获取时间戳：
```java
		
		//获取时间戳方法1
        Instant timestamp = Instant.now();
        System.out.println("Instant获取时间戳：" + timestamp.toEpochMilli());
        //获取时间戳方法2
        Clock clock = Clock.systemUTC();
        System.out.println("Clock时间戳 : " + clock.millis());
        //获取时间戳方法3
        Clock defaultClock = Clock.systemDefaultZone();
        System.out.println("defaultClock时间戳: " + defaultClock.millis());
```
无论是使用Clock.systemUTC或者Clock.systemDefaultZone获得的时间戳都是一样的，区别在于Clock.systemUTC返回的时钟的区域为UTC时区，Clock.systemDefaultZone返回的是你所在国家的时区，所以如果你还要把你的时间戳转换成LocalDateTime这样具体的年月日时分秒，那你就应该用Clock.systemDefaultZone

时间戳转LocalDateTime：
```java
		Clock defaultClock = Clock.systemDefaultZone();
        System.out.println("当前时间戳 : " + defaultClock.millis());//当前时间戳 : 1660488184650
        LocalDateTime data1 = LocalDateTime.now(defaultClock);
        System.out.println("时间戳转化为时间" + data1.toString());//时间戳转化为时间2022-08-14T22:43:04.654
```

获取巴黎的当前时间：

```java
		Clock clock = Clock.system(ZoneId.of("Europe/Paris")); //巴黎时区
        System.out.println("巴黎当前时间戳 : " + clock.millis());//巴黎当前时间戳 : 1660488329801
        LocalDateTime data=LocalDateTime.now(clock);
        System.out.println("巴黎当前时间" + data.toString());//巴黎当前时间2022-08-14T16:45:29.802
```
## 十八、LocalDateTime与时间戳的转换

LocalDateTime 转为毫秒级和秒级时间戳：
```java
        LocalDateTime localDateTime = LocalDateTime.now();
        Timestamp ts = Timestamp.valueOf(localDateTime);

        //转为毫秒级时间戳
        long milliSecondStamp = ts.getTime();

        //转为秒级时间戳
        Instant instant = ts.toInstant();
        long secondStamp =instant.getEpochSecond();

```

毫秒级和秒级时间戳转为 LocalDateTime：
```java
        //毫秒级时间戳转LocalDateTime
        long milliSecondStamp = 1681355916663L;
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDateTime localDateTime = Instant.ofEpochMilli(milliSecondStamp).atZone(zoneId).toLocalDateTime();

        //级时间戳转LocalDateTime
        long secondStamp  = 1681355916L;
        ZoneId zoneId1 = ZoneId.systemDefault();
        LocalDateTime localDateTime1 = Instant.ofEpochSecond(secondStamp).atZone(zoneId1).toLocalDateTime();

```

## 十九、LocalDateTime 与 Date 的转换
```java
		//Date 转 LocalDateTime
		Date date1 = new Date();
        LocalDateTime localDateTime1 = date1.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();

		//LocalDateTime转Date 
		LocalDateTime localDateTime2 = LocalDateTime.now();
        Date date2 = Date.from(localDateTime2.atZone(ZoneId.systemDefault()).toInstant());
```

---

参考：
[LocalDate和LocalTime的用法介绍](https://www.cnblogs.com/qingyunfc/p/10236734.html)
 [Java 8新特性（四）：新的时间和日期API](https://lw900925.github.io/java/java8-newtime-api.html)
  [Java核心之常见时间日期](https://blog.csdn.net/ruan_luqingnian/article/details/113816109?utm_medium=distribute.pc_category.none-task-blog-hot-7.nonecase&depth_1-utm_source=distribute.pc_category.none-task-blog-hot-7.nonecase&request_id=)