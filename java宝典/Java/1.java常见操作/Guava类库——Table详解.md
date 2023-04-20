[TOC]


# 前言
Guava 是一套来自 Google 的核心 Java 库，其中包括新的集合类型、不可变的集合、图库，以及并发、I/O、散列、缓存、基元、字符串等实用工具！

它更细粒度的提供了一些jdk没有的api和优化了一些操作，可以让你的代码更加优雅简洁高效。

它被广泛用于 Google 内部的大多数 Java 项目，也被许多其他公司广泛使用。它被广泛用于 Google 内部的大多数 Java 项目，也被许多其他公司广泛使用。

# 一、使用Guava
引入maven如下：
```xml
		<dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.1-jre</version>
        </dependency>
```


# 二、Table的创建和介绍
平常我们会遇到创建Map<String, Map<String, Integer>>这样两级嵌套的Map类型的代码，遍历和操作都十分不方便，为了让我们代码看起来更优雅和方便操作，所以Table就出现了。

1.以前我们的代码可能会是这样的：
```java
		Map<String, Integer> mapTemp1 = new HashMap<>();
        mapTemp1.put("age", 20);
        mapTemp1.put("score", 98);
        Map<String, Integer> mapTemp2 = new HashMap<>();
        mapTemp2.put("age", 22);
        mapTemp2.put("score", 100);
		
		Map<String, Map<String, Integer>> map = new HashMap<>();
        map.put("Tom", mapTemp1);
        map.put("Jery", mapTemp2);
		//{Jery={score=100, age=22}, Tom={score=98, age=20}}
        System.out.println(map);
```


2.Table的创建
**Table<R, C, V>中的三个参数我们简称为行、列、值，分别直观的对应两层Map中的Map<R, Map<C, V>>** 
使用Table上面的代码可简化为下面的代码。
```java
		//创建一个Table
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
```

3.如果行和列需要按照自然顺序可以使用TreeBasedTable.create()创建
```java

        Table<String, String, Integer> tab = TreeBasedTable.create();
```

4.如果打算创建一个不可变的Table，其内部数据永远不会改变（初始化后不能增加和删除数据），请使用ImmutableTable创建Table：
```java

        Table<String, String, Integer> tab = ImmutableTable.<String, String, Integer>builder()
                .put("Tom", "age", 20)
                .put("Jery", "age", 22)
                .put("Tom", "score", 98)
                .put("Jery", "score", 100)
                .build();
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
```

# 三、通过行和列来获取值
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);

        // 通过行和列来获取值
        Integer age = tab.get("Tom", "age");
		//20
        System.out.println(age);
```

# 四、通过行和列来删除值
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);

        //通过行和列删除值。返回删除的值
        Integer value = tab.remove("Tom", "score");
		//98
        System.out.println(value);
		//{Tom={age=20}, Jery={age=22, score=100}}
        System.out.println(tab);
```

# 五、判断行、列、值是否存在
```java
		Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
        //根据行和列判断值是否存在
        boolean bo1 = tab.contains("Tom", "score");
        //判断行是否存在
        boolean bo2 = tab.containsRow("Tom");
        //判断列是否存在
        boolean bo3 = tab.containsColumn("age");
        //判断值是否存在
        boolean bo4 = tab.containsValue(100);
		//true
        System.out.println(bo1);
		//true
        System.out.println(bo2);
		//true
        System.out.println(bo3);
		//true
        System.out.println(bo4);
       
```

# 六、Table的遍历
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);

        // 遍历表格
        for (Table.Cell c : tab.cellSet()) {
            System.out.println("行：" + c.getRowKey() + ",列：" + c.getColumnKey() + ",值：" + c.getValue());
        }
```

输出：
```java
{Tom={age=20, score=98}, Jery={age=22, score=100}}
行：Tom,列：age,值：20
行：Tom,列：score,值：98
行：Jery,列：age,值：22
行：Jery,列：score,值：100
```

# 七、Table转Map

1.Table<R, C, V>转Map<R, Map<C, V>>:
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		Map<String, Map<String, Integer>> rowMap = tab.rowMap();
		//{age={Tom=20, Jery=22}, score={Tom=98, Jery=100}}
        System.out.println(rowMap);
```

2.Table<R, C, V>转Map<C, Map<R, V>>:
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		Map<String, Map<String, Integer>> columnMap = tab.columnMap();
		//{age={Tom=20, Jery=22}, score={Tom=98, Jery=100}}
        System.out.println(columnMap);
```

3.Table<R, C, V>转Map<C, V>:
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		Map<String, Integer> ageMap = tab.column("age");
		//{Tom=20, Jery=22}
        System.out.println(ageMap);
```
4.根据指定的R将Table<R, C, V>转Map<C, V>:
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		//根据指定的行将列和值组成一个Map
        Map<String, Integer> tomMap = tab.row("Tom");
		//{age=20, score=98}
        System.out.println(tomMap);
```

5.根据指定的C将Table<R, C, V>转Map<R, V>:
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		//根据指定的列将行和值组成一个Map
        Map<String, Integer> ageMap = tab.column("age");
		//{Tom=20, Jery=22}
        System.out.println(ageMap);
```


# 八、Table翻转行和列
将Table<R, C, V>转为Table<C, R, V>:
```java
		Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
        //{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);

        Table<String, String, Integer> transposedTab = Tables.transpose(tab);
        //{age={Tom=20, Jery=22}, score={Tom=98, Jery=100}}
        System.out.println(transposedTab);
```

# 九、Table的行列值转为Set

1.获取所有的行为一个Set
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
        //获取所有的行为一个Set
        Set<String> rowSet = tab.rowKeySet();
		//[Tom, Jery]
        System.out.println(rowSet);
```
2.获取所有的列为一个Set
```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
		//获取所有的列为一个Set
        Set<String> columnSet = tab.columnKeySet();
		//[age, score]
        System.out.println(columnSet);
```
3.获取所有的值为一个集合

```java
        Table<String, String, Integer> tab = HashBasedTable.create();
        tab.put("Tom", "age", 20);
        tab.put("Jery", "age", 22);
        tab.put("Tom", "score", 98);
        tab.put("Jery", "score", 100);
		//{Tom={age=20, score=98}, Jery={age=22, score=100}}
        System.out.println(tab);
		
        //获取所有的值为一个集合
        Collection<Integer> values = tab.values();
		//[20, 98, 22, 100]
        System.out.println(values);
```