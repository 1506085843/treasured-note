[TOC]

# 前言 
Guava 是一套来自 Google 的核心 Java 库，其中包括新的集合类型、不可变的集合、图库，以及并发、I/O、散列、缓存、基元、字符串等实用工具！

它更细粒度的提供了一些jdk没有的api和优化了一些操作，可以让你的代码更加优雅简洁高效。

它被广泛用于 Google 内部的大多数 Java 项目，也被许多其他公司广泛使用。它被广泛用于 Google 内部的大多数 Java 项目，也被许多其他公司广泛使用。

# 一、引入Guava
maven如下：
```xml
		<dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.1-jre</version>
        </dependency>
```


# 二、BiMap
jdk自带的Map是key是唯一的，value可以是重复的。

那能不能key和value都是唯一的呢,通过key可以找到value，通过value也可以找到key。

BiMap就是这样的双向map,且key和value都是唯一的。

注意:是在使用BiMap进行put时value不能重复，否则会抛出异常。

**1.BiMap的创建和使用**
```java
		//创建一个BiMap
        BiMap<String,String> bigMap = HashBiMap.create();
        bigMap.put("a","1");
        bigMap.put("b","2");
		
        //通过key查找value
        System.out.println(bigMap.get("a"));//1
        //通过value查找key
        System.out.println(bigMap.inverse().get("2"));//b
```
**2.HashMap转为BiMap**
```java
        Map<String, String> map = new HashMap<>();
        map.put("a","1");
        map.put("b","2");

        BiMap<String,String> bigMap = HashBiMap.create(map);
        System.out.println(bigMap);//{a=1, b=2}
```
**3.使用forcePut覆盖value的值**
```java
        BiMap<String,String> bigMap = HashBiMap.create();
        bigMap.put("a","1");
        bigMap.put("b","2");
        System.out.println(bigMap);//{a=1, b=2}
        bigMap.forcePut("c","2");
        System.out.println(bigMap);//{a=1, c=2}
```

# 三、Multimap

代码里我们经常这样写 Map<k,List<v>>，好像有点臃肿！另外我们还得判断key是否存在来决定是否 new 一个 List 出来，有点麻烦！更加麻烦的事情还在后头，比如遍历，比如删除。

所以Guava产生了Multimap来代替 Map<k,List<v>> ，Multimap有点类似Map，但是Multimap中每个键可能与多个值相关联（即重复put相同的key时不会覆盖原来的）

**1.Multimap的创建和使用**
```java
		Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
		//{a=[1, 2, 2], b=[3, 4]}
        System.out.println(map);
```

**2.获取值**

(1) 根据key获取值
```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        System.out.println(map);

        Collection<Integer> values = map.get("a");
        //[1, 2, 2]
        System.out.println(values);
```

(2) 获取所有的key和value
```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        System.out.println(map);

        Set<String> keys = map.keySet();
        //[a, b]
        System.out.println(keys);
        Collection<Integer> values = map.values();
        //[1, 2, 2, 3, 4]
        System.out.println(values);
```
**3. 获取所有的key的大小和value的大小**
size()返回存储 Map 中实际值的数量，但keySet().size()返回不同键的数量。
```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        System.out.println(map);

        int keysSize = map.keySet().size();
        //2
        System.out.println(keysSize);
        int valuesSize = map.size();
        //5
        System.out.println(valuesSize);
```

**4. Multimap转Map**

Multimap转Map后，这个Map支持remove和修改操作，但是不支持put和putAll操作
```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        //{a=[1, 2, 2], b=[3, 4]}
        System.out.println(map);
        
        Map<String, Collection<Integer>> map1 = map.asMap();
        Collection<Integer> aValue = map1.get("a");
        aValue.add(999);
        //{a=[1, 2, 2, 999], b=[3, 4]}
        System.out.println(map1);

//       不能put会抛出异常        
//        Collection<Integer> collection=new ArrayList<>();
//        collection.add(88);
//        collection.add(88);
//        map1.put("c",collection);
```

**5.遍历**
```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        //{a=[1, 2, 2], b=[3, 4]}
        System.out.println(map);

        for (Map.Entry<String,Integer> m: map.entries()) {
            System.out.println("key是："+m.getKey()+" value是："+m.getValue());
        }
```

输出：
```java
{a=[1, 2, 2], b=[3, 4]}
key是：a value是：1
key是：a value是：2
key是：a value是：2
key是：b value是：3
key是：b value是：4
```

转换为Map后遍历：

```java
        Multimap<String,Integer> map = ArrayListMultimap.create();
        map.put("a",1);
        map.put("a",2);
        map.put("a",2);
        map.put("b",3);
        map.put("b",4);
        //{a=[1, 2, 2], b=[3, 4]}
        System.out.println(map);

		Map<String, Collection<Integer>> map1 = map.asMap();
        for (Map.Entry<String,Collection<Integer>> m:map1.entrySet()) {
            System.out.println("key是："+m.getKey()+" value是："+m.getValue());
        }
```
输出：
```java
{a=[1, 2, 2], b=[3, 4]}
key是：a value是：[1, 2, 2]
key是：b value是：[3, 4]
```

**6.使用Multimap的优点**
- 在使用put()添加数据之前无需创建新的空集合
- get() 方法不返回null，只返回一个空集合（可以不用像Map<k,List<v>>获取值时去判空检查）