[TOC]

# 前言
关于stream流的操作网上有很多教程，比如流里的 filter、map、flatMap、collect、reduce、limit、distinct、skip、findFirst、count等常用操作。

如对基本概念不熟，可参考其他博客：
[JAVA8 Stream接口，map操作，filter操作，flatMap操作](https://blog.csdn.net/qq_28410283/article/details/80642786)
[java8 stream流操作的flatMap（流的扁平化）](https://blog.csdn.net/Mark_Chao/article/details/80810030?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)
[Java 8 Stream 的终极技巧——Collectors 操作](https://www.cnblogs.com/felordcn/p/collectors.html)

本文主要讲开发中常用的流操作，熟练使用可简化代码、使代码更简洁优雅。

---

# 一、  List\<String\> 转 List\<Integer\>
如果 strList 中全是数字字符串，可通过如下转换为 integerList 
```java
        List<String> strList = new ArrayList<>(Arrays.asList("7","8","9"));
        List<Integer> integerList = strList.stream().map(Integer::valueOf).collect(Collectors.toList());
```
# 二、  List\<String\> 转 Integer[ ]
如果 strList 中全是数字字符串，可通过如下转换为 intergerArry 

```java
        List<String> strList = new ArrayList<>(Arrays.asList("7","8","9"));
        Integer[] intergerArry = strList.stream().map(Integer::valueOf).toArray(Integer[]::new);
```

# 三、 String 转 List\<Character\>
```java
        String str = "abcdef";
        List<Character> list = str.chars().mapToObj(x -> (char) x).collect(Collectors.toList());
```
# 四、 String[ ] 转 List\<String>

```java
        String[] array = new String[]{"aa","bb","cc"};
        List<String> strList = Arrays.stream(array).collect(Collectors.toList());
```

# 五、 String[ ] 转  Integer [ ] 

```java
        String[] strArry = new String[]{"5", "6", "1", "4", "9"};
        Integer[] integerArry = Arrays.stream(strArry).map(Integer::parseInt).toArray(Integer[]::new);
```

# 六、 List\<Integer\>  转 int [ ]
```java
        List<Integer> integerList = new ArrayList<>(Arrays.asList(5,6,1,4,9));
        int[] arr = integerList.stream().mapToInt(Integer::intValue).toArray();
```

# 七、 int [ ] 转 List\<Integer\>
```java
        int[] intArry = new int[]{5,6,1,4,9};
        List<Integer> integerList = Arrays.stream(intArry).boxed().collect(Collectors.toList());
```

# 八、int [ ] 转 Integer [ ] 

```java
        int[] intArry = new int[]{5, 6, 1, 4, 9};
        Integer[] integerArry = Arrays.stream(intArry).boxed().toArray(Integer[]::new);
```


# 九、int [] 最大、最小值、平均值，求和

```java
        int[] arr = new int[]{12,3,34,67,100,99};
        int maxValue = Arrays.stream(arr).max().getAsInt();
        int minValue = Arrays.stream(arr).min().getAsInt();
        double averValue = Arrays.stream(arr).average().getAsDouble();
        int sumValue = Arrays.stream(arr).sum();
        System.out.println("the max:" + maxValue);
        System.out.println("the min:" + minValue);
        System.out.println("the average:" + averValue);
        System.out.println("the sum:" + sumValue);
```
上面是流里面使用 max、min、average 、sum方法获取数组的最大、最小值、平均值，求和。
除了上面的方法还可以像下面这样获取最大值、最小值、平均值，求和。
```java
        int[] intArray = {12,3,34,67,100,99};
        IntStream intStream = IntStream.of(intArray);
        IntSummaryStatistics statistics = intStream.summaryStatistics();
        System.out.println("the max:" + statistics.getMax());
        System.out.println("the min:" + statistics.getMin());
        System.out.println("the average:" + statistics.getAverage());
        System.out.println("the sum:" + statistics.getSum());
        System.out.println("the count:" + statistics.getCount());//statistics.getCount()相当于获取数组大小
```

# 十、String[] 对每个元素分割并转换为其他类型

```java
        String[] array = {"a-1", "b-2", "c-3"};

        //"-分割后获取字母，转为新数组
        String[] strArray = Arrays.stream(array).map(v -> v.split("-")[0]).toArray(String[]::new);
        //"-分割后获取字母，转List
        List<String> list = Arrays.stream(array).map(v -> v.split("-")[0]).collect(Collectors.toList());
        //"-分割后获取数字，转List
        List<Integer> numberlist = Arrays.stream(array).map(v -> Integer.parseInt(v.split("-")[1])).collect(Collectors.toList());
        //"-分割后获取字母，用逗号拼接为字符串
        String str = Arrays.stream(array).map(v -> v.split("-")[0]).collect(Collectors.joining(","));

        System.out.println(Arrays.toString(strArray));//[a, b, c]
        System.out.println(list);//[a, b, c]
        System.out.println(numberlist);//[1, 2, 3]
        System.out.println(str);//a,b,c
```

# 十一、List\<String\> 逗号拼接为一个字符串
```java
        List<String> strList = new ArrayList<>(Arrays.asList("a","b","c"));
        String str = strList.stream().collect(Collectors.joining(","));
        System.out.println(str);//a,b,c
```
# 十二、List\<Integer\> 逗号拼接为一个字符串
```java
        List<Integer> integerList = Arrays.asList(7, 8, 9);
        String str = integerList.stream().map(String::valueOf).collect(Collectors.joining(","));
        System.out.println(str);//7,8,9
```
# 十三、生成指定范围的数组或List
- 生成[0,100)的 int[ ]

```java
        int[] intArray = IntStream.range(0, 100).toArray();
```
- 生成[0,100)的 List

```java
		List<Integer> intList= IntStream.range(0, 100)
                .boxed()
                .collect(toList());
                
        List<String> strList = IntStream.range(0, 100)
                .boxed()
                .map(i -> i.toString())
                .collect(toList());
```
- 生成[0,100]的 int[ ]

```java
        int[] intArray1 = IntStream.rangeClosed(0, 100).toArray();
```
- 生成[0,100]的 List

```java
		List<Integer> intList= IntStream.rangeClosed(0, 100)
                .boxed()
                .collect(toList());
                
        List<String> strList = IntStream.rangeClosed(0, 100)
                .boxed()
                .map(i -> i.toString())
                .collect(toList());
```
# 十四、判断数组中是否含有某一值
- 字符串数组
```java
        String[] values = {"AB","BC","CD","AE"};
        boolean contains = Arrays.stream(values).anyMatch("AE"::equals);
```
- int数组
```java
        int[] a = {1,2,3,4};
        boolean contains = IntStream.of(a).anyMatch(x -> x == 4);
```
# 十五、数组或List求和
- 数组求和
```java
        int[] intArray = {11, 5, 3, 2, 1};
        int sum = Arrays.stream(intArray).reduce(0, Integer::sum);
```
- List求和
```java
        List<Integer> list = new ArrayList<>(Arrays.asList(5, 1, 7, 10));
        int sum = list.stream().reduce(0, Integer::sum);
```
# 十六、两个数组合并为一个新的数组
- 两个字符串数组合并为一个新的数组

```java
        String[] a = {"a", "b", "c"};
        String[] b = {"1", "2", "3"};
        String[] c = Stream.of(a,b).flatMap(Stream::of).toArray(String[]::new);
```
- 两个 int 型数组合并为一个新的 int 型数组

```java
        int[] a = new int[]{1,3};
        int[] b = new int[]{2,4};
        int[] c =  IntStream.concat(Arrays.stream(a), Arrays.stream(b)).toArray();
```
# 十七、两个数组合并为一个List
- 两个String数组转List\<String\>

```java
        String[] a = {"a", "b", "c"};
        String[] b = {"1", "2", "3"};
        List<String> strList = Stream.of(a, b).flatMap(Stream::of).collect(Collectors.toList());
```
- 两个int数组转List\<Integer\>

```java
        int[] a = new int[]{1, 2};
        int[] b = new int[]{3, 4};
        List<Integer> integerList = Stream.of(IntStream.of(a).boxed(), IntStream.of(b).boxed()).flatMap(s -> s).collect(Collectors.toList());
```
# 十八、两个List合并为一个新的List
```java
        List<String> lis1 = new ArrayList<>(Arrays.asList("a", "b", "c"));
        List<String> list2 = new ArrayList<>(Arrays.asList("e", "f", "g"));
        List<String> newList = Stream.of(lis1, list2).flatMap(Collection::stream).collect(Collectors.toList());
```

# 十九、List\<Integer\>求交集、并集、差集
```java
List<Integer> list = new ArrayList<>(Arrays.asList(7, 8, 9));
List<Integer> list2 = new ArrayList<>(Arrays.asList(3,4, 9));
```
对 list 和 list2求交集、并集、差集

```java
//交集
 List<Integer> beMixed = list.stream().filter(list2::contains).collect(Collectors.toList());
 System.out.println(beMixed);//[9]

//并集
List<Integer> aggregate = Stream.of(list, list2).flatMap(Collection::stream).distinct().collect(Collectors.toList());
System.out.println(aggregate);//[7, 8, 9, 3, 4]

//差集
List<Integer> subtraction = list.stream().filter(v->!list2.contains(v)).collect(Collectors.toList());
System.out.println(subtraction);//[7, 8]
```

# 二十、Map的value保存为List、Set
- 保存为List
```java
        Map<String, String> map = new HashMap<>();
        map.put("1", "a");
        map.put("2", "b");
        map.put("3", "c");
        List<String> values = map.values().stream().collect(Collectors.toList());
```
- 保存为Set
```java
        Map<String, String> map = new HashMap<>();
        map.put("1", "a");
        map.put("2", "b");
        map.put("3", "c");
        Set<String> values = map.values().stream().collect(Collectors.toSet());
```

# 二十一、Map对value求和
```java
        Map<String, Integer> map = new HashMap<>();
        map.put("1", 11);
        map.put("2", 22);
        map.put("3", 100);
        Integer sum = map.values().stream().mapToInt(Integer::valueOf).sum();
        System.out.println(sum);
```


# 二十二、Map<String, List<Integer\>>获取所有的values为一个List<Integer\>
将map 里的所有value合并为一个List
```java
       List<Integer> list1 = new ArrayList<>(Arrays.asList(1, 2, 3));
        List<Integer> list2 = new ArrayList<>(Arrays.asList(4, 5, 6));
        List<Integer> list3 = new ArrayList<>(Arrays.asList(7, 8, 9));

        Map<String, List<Integer>> map = new HashMap<>();
        map.put("a", list1);
        map.put("b", list2);
        map.put("c", list3);

        List<Integer> list = map.values().stream().flatMap(Collection::stream).collect(Collectors.toList());
        //下面的代码是一样的效果
        //List<Integer> list = map.entrySet().stream().map(e -> e.getValue()).flatMap(Collection::stream).collect(Collectors.toList());
        System.out.println(list);//[1, 2, 3, 4, 5, 6, 7, 8, 9]
```


# 二十三、String[]中的元素转大写并转为List\<String\>
```java
        String[] strArray= { "java", "react", "angular", "vue" };
        List<String> list = Stream.of(strArray).map(test -> test.toUpperCase()).collect(Collectors.toList());
```

# 二十四、对String字符串中的数字求和
首先需要将该字符串转换为数组，接下来需要过滤掉非整数元素，最后，将该数组的剩余元素转换为数字并求和。
```java
        String string = "Item1 10 Item2 25 Item3 30 Item4 45";

        Integer sum = Arrays.stream(string.split(" "))
                .filter((s) -> s.matches("\\d+"))
                .mapToInt(Integer::valueOf)
                .sum();
```
# 二十五、List\<String\>中统计首字母是j的个数
```java
        List<String> list = Arrays.asList("java", "react", "angular", "javascript", "vue");
        long count = list.stream().filter(p -> p.startsWith("j")).count();
```
# 二十六、List\<String\>中获取第一个首字母是j的元素
```java
        List<String> list = Arrays.asList( "react","java", "angular", "javascript", "vue");
        //java
        String firstJ = list.stream().filter(p -> p.startsWith("j")).findFirst().get();
```
# 二十七、List\<String\>统计各字符串出现的次数

```java
        List<String> items = Arrays.asList("apple", "apple", "orange", "orange", "orange", "blueberry", "peach", "peach", "peach", "peach");
        Map<String, Long> result = items.stream().collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
        //{orange=3, apple=2, blueberry=1, peach=4}
        System.out.println(result);
```

# 二十八、 List去除重复元素
```java
        List<String> list = Arrays.asList("aa", "bb", "cc", "bb");
        List<String> distinctList = list.stream().distinct().collect(Collectors.toList());
```

# 二十九、List<List\<String\>>转List\<String\>
```java
        List<String> a = Arrays.asList("Virat", "Dhoni", "Jadeja");
        List<String> b = Arrays.asList("1", "2", "3");
        List<List<String>> list = new ArrayList<>();
        list.add(a);
        list.add(b);

        List<String> newList = list.stream().flatMap(v -> v.stream()).collect(Collectors.toList());
        //[Virat, Dhoni, Jadeja, 1, 2, 3]
        System.out.println(newList);
```

# 三十、多个List\<String\> 合并为一个List\<String\>
```java
    public static void main(String[] args) {
        List<String> list1=new ArrayList<>(Arrays.asList("a","b","c"));
        List<String> list2=new ArrayList<>(Arrays.asList("d","e","f"));
        List<String> list3=new ArrayList<>(Arrays.asList("g","h","i"));
        List<String> list4=new ArrayList<>(Arrays.asList("1","2"));
        List<String> list5=new ArrayList<>(Arrays.asList("3","4","5"));

        List<String> result1 = join(list1, list2,list3);
        List<String> result2 = join(list4, list5);
        //[a, b, c, d, e, f, g, h, i]
        System.out.println(Arrays.toString(result1.toArray()));
        //[1, 2, 3, 4, 5]
        System.out.println(Arrays.toString(result2.toArray()));
    }

    @SafeVarargs
    public static  List<String> join(List<String>... lists) {
        return Arrays.stream(lists).flatMap(Collection::stream).collect(Collectors.toList());
    }
```



# 三十一、List\<Object\>转Map<String, Object>
假设有Student类，里面含有id、name、score等信息。现在想把 List\<Student\> 转化为Map<String,Student>，其中key是name。
```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "张三", 85));
        list.add(new Student(2, "李四", 60));
        Map<String, Student> map = list.stream().collect(Collectors.toMap(Student::getName, Function.identity()));
```
# 三十二、 List\<Object\>针对某一成员变量获取最大、小值的Object
假设有 Student 类，里面含有 id、name、score 等信息。现在想获取List\<Student\> 中 score 最大的 Student 和 score 最小的 Student。

```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "张三", 85));
        list.add(new Student(2, "李四", 60));
        list.add(new Student(3, "刘一", 70));
        list.add(new Student(4, "李四", 99));
        //获取score最大的Student
        Student maxStudent = list.stream().max(Comparator.comparing(Student::getScore)).get();
        //获取score最小的Student
        Student minxStudent = list.stream().min(Comparator.comparing(Student::getScore)).get();
```
# 三十三、 List\<Object\>获取某个成员变量最大、最小值、平均值，求和
假设有 Student 学生类，里面含有 id、name、score 等信息。现在想获取 List\<Student\> 中的 score 的最大值、最小值、平均值，求和：
```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "小名", 17));
        list.add(new Student(2, "小红", 18));
        list.add(new Student(3, "小蓝", 19));
        list.add(new Student(4, "小灰", 20));
        list.add(new Student(5, "小黄", 21));
        list.add(new Student(6, "小白", 22));
        
        IntSummaryStatistics intSummary = list.stream().collect(Collectors.summarizingInt(Student::getScore));
        System.out.println(intSummary.getAverage());// 19.5
        System.out.println(intSummary.getMax());// 22
        System.out.println(intSummary.getMin());// 17
        System.out.println(intSummary.getSum());// 117
```
如果你只想求和可以像下面这样写：
```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "小名", 17));
        list.add(new Student(2, "小红", 18));
        list.add(new Student(3, "小蓝", 19));
        list.add(new Student(4, "小灰", 20));
        list.add(new Student(5, "小黄", 21));
        list.add(new Student(6, "小白", 22));

        Integer sum = list.stream().mapToInt(Student::getScore).sum();//117
        //下面注释的两行代码和上一行的效果一样，都可以进行求和
        //Integer sum = list.stream().map(Student::getScore).reduce(0, Integer::sum);
        //Integer sum = list.stream().map(Student::getScore).mapToInt(Integer::intValue).sum();
```
# 三十四、List\<Integer> 获取最大、最小值、平均值，求和

```java
        List<Integer> list = Arrays.asList(12, 3, 34, 67, 100, 99);
        int maxValue = list.stream().max(Integer::compare).get();
        int minValue = list.stream().min(Integer::compare).get();
        double averValue = list.stream().mapToDouble(a -> a).average().getAsDouble();
        int sumValue = list.stream().reduce(0, Integer::sum);
        System.out.println("the max:" + maxValue);
        System.out.println("the min:" + minValue);
        System.out.println("the average:" + averValue);
        System.out.println("the sum:" + sumValue);
```
上面是流里面使用 max、min、average 、reduce 方法获取数组的最大、最小值、平均值，求和。
除了上面的方法还可以像下面这样获取最大值、最小值、平均值，求和。

```java
        List<Integer> list = new ArrayList<>(Arrays.asList(12,3,34,67,100,99));
        IntSummaryStatistics statistics = list.stream().mapToInt(value -> value).summaryStatistics();
        System.out.println("the max:" + statistics.getMax());
        System.out.println("the min:" + statistics.getMin());
        System.out.println("the average:" + statistics.getAverage());
        System.out.println("the sum:" + statistics.getSum());
        System.out.println("the count:" + statistics.getCount());//获取list大小
```


拓展：
除了anyMatch ，stream 中还有 allMatch、noneMatch ，可以参考 [java8新特性Stream流中anyMatch和allMatch和noneMatch的区别详解](https://blog.csdn.net/yuxiangdeming/article/details/121288780)
