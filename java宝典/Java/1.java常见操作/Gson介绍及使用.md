[TOC]


# 前言
 Gson 是一个由谷歌开发的基于 Java 的简单库，用于将 Java 对象序列化为 JSON，或者将 JSON 转化为 Java 对象。它

Gson库的优点使用这个库 -

- **标准化**- Gson 是由谷歌管理的标准化库。
- **高效**- 它是对 Java 标准库的可靠、快速和高效的扩展。
- **Optimized** - 该库是高度优化的。
- **Support Generics** - 它为泛型提供广泛的支持。
- **Supports complex inner classes** - 它支持具有深继承层次结构的复杂对象。


# 一、导入Gson依赖
maven如下：
```xml
<dependency>
  <groupId>com.google.code.gson</groupId>
  <artifactId>gson</artifactId>
  <version>2.10.1</version>
</dependency>
```
# 二、json对象与json数组的创建
## json对象创建
```java
        JsonObject info = new JsonObject();
        info.addProperty("name", "张三");
        info.addProperty("age", "18");
        info.addProperty("地理", 70);
        info.addProperty("英语", 60);
```
## json数组创建
```java
        JsonObject info = new JsonObject();
        info.addProperty("name", "张三");
        info.addProperty("age", "18");
        info.addProperty("地理", 70);
        info.addProperty("英语", 60);
        System.out.println(info);

        JsonObject info1 = new JsonObject();
        info1.addProperty("name", "张三");
        info1.addProperty("age", "18");
        JsonObject info2 = new JsonObject();
        info2.addProperty("name", "李四");
        info2.addProperty("age", "19");

        //把上面创建的两个json对象加入到json数组里
        JsonArray array = new JsonArray();
        array.add(info1);
        array.add(info2);
```

```java
        JsonArray array = new JsonArray();
        array.add("1班");
        array.add("2班");
        array.add("3班");
```

# 三、json对象取值与json数组遍历取值
## json对象取值
```java
        JsonArray array = new JsonArray();
        array.add("1班");
        array.add("2班");
        array.add("3班");

        JsonObject school = new JsonObject();
        school.addProperty("schoolName", "第一中学");
        school.addProperty("teacher", "刘梅");

        JsonObject info = new JsonObject();
        info.addProperty("name", "张三");
        info.addProperty("age", 18);
        info.add("gradle",array);
        info.add("schoolInfo",school);

        //从info中取值
        System.out.println(info.get("name").getAsString()); //张三
        System.out.println(info.get("age").getAsInt());//18
        System.out.println(info.getAsJsonArray("gradle"));//["1班","2班","3班"]
        System.out.println(info.getAsJsonObject("schoolInfo"));//{"schoolName":"第一中学","teacher":"刘梅"}
```

## json数组遍历取值
```java
        JsonObject info1 = new JsonObject();
        info1.addProperty("name", "张三");
        info1.addProperty("age", 18);
        JsonObject info2 = new JsonObject();
        info2.addProperty("name", "李四");
        info2.addProperty("age", 19);

        JsonArray array = new JsonArray();
        array.add(info1);
        array.add(info2);
        //遍历获取json数组中对象的值
        for (int i = 0; i < array.size(); i++) {
            JsonObject  json = array.get(i).getAsJsonObject();
            System.out.println(json.get("name").getAsString());
            System.out.println(json.get("age").getAsInt());
        }

//或者：
//        for (JsonElement element : array) {
//            JsonObject  json = element.getAsJsonObject();
//            System.out.println(json.get("name").getAsString());
//            System.out.println(json.get("age").getAsInt());
//        }
```

```java
        JsonArray array = new JsonArray();
        array.add("张三");
        array.add("李四");
        array.add("王五");

        for (JsonElement datum : array) {
            String name = datum.getAsString();
            System.out.println(name);
        }
```

# 四、json对象与字符串的转换

## json对象与字符串的转换
```java
        JsonObject info = new JsonObject();
        info.addProperty("name", "张三");
        info.addProperty("age", "18");
        info.addProperty("地理", 70);
        info.addProperty("英语", 60);

        //JSON 对象转字符串
        String str = info.toString();
        //字符串转 JSON 对象
        JsonObject jsonObject = JsonParser.parseString(str).getAsJsonObject();
```

## json字符串的字节数组转json对象
```java
        String str = "{\"name\":\"张三\",\"age\":\"18\",\"地理\":\"70\",\"英语\":\"60\"}";

        byte[] bytes = str.getBytes();
        JsonObject data = JsonParser.parseString(new String(bytes)).getAsJsonObject();
```

# 五、json数组与字符串的转换
```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串转json数组
        JsonArray data = JsonParser.parseString(str).getAsJsonArray();
        //json数组转json字符串
        String strData = data.toString();
```

# 六、json字符串数组与数组和List的转换

```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串数组转数组
        Gson gson = new Gson();
        String[] array = gson.fromJson(str, String[].class);
```

```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串数组转List
        TypeToken<List<String>> type = new TypeToken<List<String>>(){};
        List<String> list = new Gson().fromJson(str, type);
```

# 七、json字符串与java对象的转换（序列化与反序列化）

Student类如下：

```java
public class Student {
    public String name;
    public int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

```
 json字符串与java对象的转换:
```java
        Student student = new Student("张三", 18);
		Gson gson = new Gson();

        //Student对象转JSON字符串
        String studentStr = gson.toJson(student);
        //JSON字符串转Student对象
        Student student2 = gson.fromJson(studentStr, Student.class);
```

# 八、泛型类型的序列化与反序列化

上面讲的是普通对象的序列化与反序列化，但如果你的类带有泛型就不能像上面那样序列化和反序列化了。

Student类如下：

```java

public class Student<T> {
    public String name;
    public T age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public T getAge() {
        return age;
    }

    public void setAge(T age) {
        this.age = age;
    }

    public Student(String name, T age) {
        this.name = name;
        this.age = age;
    }
}
```

对泛型类型序列化与反序列化:

```java
        Student<Integer> student = new Student<Integer>("张三", 18);
        Gson gson = new Gson();

        //Student对象转JSON字符串
        Type type = new TypeToken<Student<Integer>>(){}.getType();
        String studentStr = gson.toJson(student,type);

        //JSON字符串转Student对象
        Student<Integer> student2 = gson.fromJson(studentStr, type);

```

# 九、json字符串与Map转换

## json字符串转Map
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";
        Gson gson = new Gson();
        TypeToken<Map<String, Object>> mapType = new TypeToken<Map<String, Object>>(){};
        Map<String, Object> map = gson.fromJson(str, mapType);
        System.out.println(map.get("gradle").toString());
        System.out.println(map.get("number").toString());
        System.out.println(map.get("people").toString());
```

## Map转json字符串
```java
        Gson gson = new GsonBuilder().serializeNulls().create();
        Map<String, String> map = new HashMap<>();
        map.put(null, "aa");
        map.put("测试1", null);
        map.put("测试2", "222");

        String json = gson.toJson(map);
		//{"null":"aa","测试2":"222","测试1":null}
        System.out.println(json);
```
(注意：如果直接使用 Gson gson = new Gson(); 转换，因为"测试1"的值为null，转换的结果就会是{"null":"aa","测试2":"222"}  ，所以转换为字符串时，如果你不想忽略 value 是 null 的，可以使用 Gson gson = new GsonBuilder().serializeNulls().create();)

# 十、json数组转List

json数组转List<Map<String, String>> ：
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";

        //字符串转JSON对象
        JsonObject jsonObject =JsonParser.parseString(str).getAsJsonObject();
        
        //获取people数组
        //JsonElement people = jsonObject.get("people");
        JsonArray people = jsonObject.get("people").getAsJsonArray();
        
        //json数组转List
        TypeToken<List<Map<String, String>>> type = new TypeToken<List<Map<String, String>>>(){};
        List<Map<String, String>> peopleList = new Gson().fromJson(people, type);
```
json字符串数组转List ：

```java
String str="[{\"name\":\"张三\",\"age\":\"18\"}, {\"name\":\"李四\",\"age\":\"19\"}]";

//json字符串数组转List
TypeToken<List<Student>> type = new TypeToken<List<Student>>(){};
List<Student> list = new Gson().fromJson(str, type);
```

# 十一、json字符串格式化

有时候我们想把 json 字符串格式化输出，也就是该缩进的缩进该换行的换行，让它更美观的输出，可以像下面这样：

```java
        String str = "[{\"isSendPhone\":\"true\",\"id\":\"22258352\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"7\",\"count\":\"5\",\"serviceName\":\"流程助手\",\"startdate\":\"2022-02-09 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-08 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"xx有限责任公司\",\"serviceId\":\"21\",\"isSendeMail\":\"true\",\"email\":\"\"},{\"isSendPhone\":\"true\",\"eid\":\"7682130\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"15\",\"count\":\"50\",\"serviceName\":\"经理人自助服务\",\"startdate\":\"2022-02-17 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-16 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"xx科技股份有限公司\",\"serviceId\":\"2\",\"isSendeMail\":\"true\",\"email\":\"\"}]";
        
        JsonElement jsonObject = JsonParser.parseString(str);
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        String formatStr = gson.toJson(jsonObject);
        System.out.println(formatStr);

```

输出结果：

```java
[
  {
    "isSendPhone": "true",
    "id": "22258352",
    "phoneMessgge": "为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
    "readsendtime": "9",
    "countdown": "7",
    "count": "5",
    "serviceName": "流程助手",
    "startdate": "2022-02-09 00:00:00.0",
    "insertTime": "2023-02-02 07:00:38.0",
    "enddate": "2023-02-08 23:59:59.0",
    "emailMessage": "为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
    "phone": "",
    "companyname": "xx有限责任公司",
    "serviceId": "21",
    "isSendeMail": "true",
    "email": ""
  },
  {
    "isSendPhone": "true",
    "eid": "7682130",
    "phoneMessgge": "为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
    "readsendtime": "9",
    "countdown": "15",
    "count": "50",
    "serviceName": "经理人自助服务",
    "startdate": "2022-02-17 00:00:00.0",
    "insertTime": "2023-02-02 07:00:38.0",
    "enddate": "2023-02-16 23:59:59.0",
    "emailMessage": "为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
    "phone": "",
    "companyname": "xx科技股份有限公司",
    "serviceId": "2",
    "isSendeMail": "true",
    "email": ""
  }
]
```

---

参考：
[Gson用户指南](https://github.com/google/gson/blob/main/UserGuide.md)