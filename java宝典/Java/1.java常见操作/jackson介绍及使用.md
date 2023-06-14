[TOC]


# 前言
Jackson 于 2009 年 5 月首次正式发布，旨在满足快速、正确和轻量级三大宗旨。Jackson 是一个成熟稳定的库，它提供了多种不同的方法来处理 JSON，包括在一些简单的用例中使用注释。

Jackson 提供了三个核心模块。

- Streaming ( `jackson-core`) 定义了一个低级流式 API 并包括特定于 JSON 的实现。
- Annotations ( `jackson-annotations`) 包含标准的 Jackson 注释。
- Databind( `jackson-databind`) 实现数据绑定和对象序列化。

将`databind`模块添加到项目中还会将流式处理和注释模块添加为传递依赖项。

下面的示例将重点关注这些核心模块；还有许多与 Jackson 一起工作的扩展和工具，这里不再赘述。

[jackson的github官网](https://github.com/FasterXML/jackson)


# 一、导入jackson依赖
maven如下：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.15.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.15.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```
# 二、json对象与json数组的创建
## json对象创建
```java
        ObjectMapper mapper = new ObjectMapper();
        ObjectNode jsonObject  =  mapper.createObjectNode();
        jsonObject.put("姓名", "张三");
        jsonObject.put("年龄", "18");
        jsonObject.put("地理", 70);

		//{"姓名":"张三","年龄":"18","地理":70}
		System.out.println(jsonObject);
```
## json数组创建

下面提供三个简单示例

```java
        //创建JSON数组
        ObjectMapper mapper = new ObjectMapper();
        ArrayNode jsonArray = mapper.createArrayNode();

        //向json数组中添加json对象
        ObjectNode jsonObject1  =  mapper.createObjectNode();
        jsonObject1.put("姓名", "张三");
        jsonObject1.put("年龄", "18");
        jsonObject1.put("地理", 70);
        jsonArray.add(jsonObject1);

        //向json数组中添加json对象
        ObjectNode jsonObject2  =  mapper.createObjectNode();
        jsonObject2.put("姓名", "李四");
        jsonObject2.put("年龄", "19");
        jsonObject2.put("地理", 80);
        jsonArray.add(jsonObject2);
        
//[{"姓名":"张三","年龄":"18","地理":70},{"姓名":"李四","年龄":"19","地理":80}]
        System.out.println(jsonArray);

```

```java
        //创建JSON数组
        ObjectMapper mapper = new ObjectMapper();
        ArrayNode jsonArray = mapper.createArrayNode();
 		//向json数组中添加数据
        jsonArray.add("张三");
        jsonArray.add("李四");

		//["张三","李四"]
        System.out.println(jsonArray);
```

```java
		//创建JSON对象     
		ObjectMapper mapper = new ObjectMapper();
        ObjectNode rootNode  =  mapper.createObjectNode();

		//创建新的JSON对象并将其添加到 rootNode json对象里
        ObjectNode child1  =  mapper.createObjectNode();
        child1.put("姓名", "张三");
        child1.put("年龄", "18");
        child1.put("地理", 70);
        rootNode.set("学生1", child1);
		//创建新的JSON对象并将其添加到 rootNode json对象里
        ObjectNode child2  =  mapper.createObjectNode();
        child2.put("姓名", "李四");
        child2.put("年龄", "19");
        child2.put("地理", 80);
        rootNode.set("学生2", child1);
		//创建新的JSON数组
        ArrayNode arrayNode = rootNode.putArray("所有学生");
        arrayNode.add("李四");
        arrayNode.add("张三");

//{"学生1":{"姓名":"张三","年龄":"18","地理":70},"学生2":{"姓名":"张三","年龄":"18","地理":70},"所有学生":["李四","张三"]}
        System.out.println(rootNode);
```

# 三、json对象取值与json数组遍历取值

## json对象取值
```java
        ObjectMapper mapper = new ObjectMapper();
        //创建JSON对象
        ObjectNode jsonObject  =  mapper.createObjectNode();
        jsonObject.put("姓名", "张三");
        jsonObject.put("年龄", 18);
        jsonObject.put("地理", 70);
        //创建JSON数组
        ArrayNode arrayNode = jsonObject.putArray("所有学生");
        arrayNode.add("李四");
        arrayNode.add("张三");

        //分别从json对象里获取姓名、年龄、所有学生
        String name = jsonObject.get("姓名").asText();
        int age = jsonObject.get("年龄").asInt();
        JsonNode  allStudents = jsonObject.get("所有学生");
        
        System.out.println(name); //张三
        System.out.println(age); //18
        System.out.println(allStudents); //["李四","张三"]
```

## json数组遍历取值
```java
        //创建JSON数组
        ObjectMapper mapper = new ObjectMapper();
        ArrayNode jsonArray = mapper.createArrayNode();
        //向json数组中添加json对象
        ObjectNode jsonObject1  =  mapper.createObjectNode();
        jsonObject1.put("姓名", "张三");
        jsonObject1.put("年龄", "18");
        jsonObject1.put("地理", 70);
        jsonArray.add(jsonObject1);
        //向json数组中添加json对象
        ObjectNode jsonObject2  =  mapper.createObjectNode();
        jsonObject2.put("姓名", "李四");
        jsonObject2.put("年龄", "19");
        jsonObject2.put("地理", 80);
        jsonArray.add(jsonObject2);
        
        //遍历获取json数组中对象的值
        for (int i = 0; i < jsonArray.size(); i++) {
            JsonNode  json = jsonArray.get(i);
            System.out.println(json.get("姓名").asText());
            System.out.println(json.get("年龄").asText());
            System.out.println(json.get("地理").asInt());
        }

//或者：
//        for (JsonNode element : jsonArray) {
//            System.out.println(element.get("姓名").asText());
//            System.out.println(element.get("年龄").asText());
//            System.out.println(element.get("地理").asInt());
//        }
```

```java
        //创建JSON数组
        ObjectMapper mapper = new ObjectMapper();
        ArrayNode jsonArray = mapper.createArrayNode();
        //向json数组中添加数据
        jsonArray.add("张三");
        jsonArray.add("李四");

        for (JsonNode element : jsonArray) {
            String name = element.asText();
            System.out.println(name);
        }
```

# 四、json对象与字符串的转换

```java
        ObjectMapper mapper = new ObjectMapper();
        ObjectNode jsonObject  =  mapper.createObjectNode();
        jsonObject.put("姓名", "张三");
        jsonObject.put("年龄", "18");
        jsonObject.put("地理", 70);

 		//JSON 对象转字符串
		String str = jsonObject.toString();

        //字符串转 JSON 对象
        ObjectNode jsonObjectNew = (ObjectNode) mapper.readTree(str);
```

# 五、json数组与字符串的转换
```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        ObjectMapper mapper = new ObjectMapper();
		//字符串转json数组
        ArrayNode jsonArray = (ArrayNode)mapper.readTree(str);
		//json数组转字符串
		String s = jsonArray.toString();
```

# 六、json字符串数组与数组和List的转换

```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串数组转数组
        ObjectMapper mapper = new ObjectMapper();
        String[] array = mapper.readValue(str, String[].class);
```

```java
        String str = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串数组转List
        ObjectMapper mapper = new ObjectMapper();
        List<String> list = mapper.readValue(str, new TypeReference<List<String>>(){});
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
}

```
 json字符串与java对象的转换:
```java
        Student student = new Student();
        student.setName("张三");
        student.setAge(18);

        ObjectMapper mapper = new ObjectMapper();

        //Student对象转JSON字符串
        String studentStr = mapper.writeValueAsString(student);
        //JSON字符串转Student对象
        String str = "{\"name\":\"张三\",\"age\":18}";
        Student studentNew = new ObjectMapper().readValue(str, Student.class);
```

从本地或网络文件中转化（反序列化）为java对象：

```java
        ObjectMapper mapper = new ObjectMapper();
        Student student = mapper.readValue(new File("F:\\json\\json1.json"), Student.class);
// 或者 从网络文件中:
        student = mapper.readValue(new URL("http://some.com/api/entry.json"), Student.class);

```

将java对象序列化为字节数组或写入文件中：

```java
        Student student = new Student();
        student.setName("张三");
        student.setAge(18);

        ObjectMapper mapper = new ObjectMapper();
       

        byte[] jsonBytes = mapper.writeValueAsBytes(student);
		// 写入文件:
         mapper.writeValue(new File("F:\\json\\json1.json"), student);
```

# 八、json字符串与Map转换

## json字符串转Map
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";
        ObjectMapper mapper = new ObjectMapper();
        Map<String, Object> map = mapper.readValue(str, Map.class);
//或者：
//Map<String, Object> map = mapper.readValue(str, new TypeReference<Map<String, Object>>() {});
```

## Map转json字符串
```java
        Map<String, String> map = new HashMap<>();
        map.put("测试1", null);
        map.put("测试2", "222");
        ObjectMapper mapper = new ObjectMapper();
        String str = mapper.writeValueAsString(map);
```
(注意：如果如果map的key有null的转换会报错)

# 十、json数组转List

json数组转List<Map<String, String>> ：
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";

        ObjectMapper mapper = new ObjectMapper();

        //字符串转JSON对象
        ObjectNode jsonObject = (ObjectNode) mapper.readTree(str);

        //获取people数组
        JsonNode people = jsonObject.get("people");

        //json数组转List
        ObjectReader reader = mapper.readerFor(new TypeReference<List<Map<String, String>>>() {});
        List<String> list = reader.readValue(people);
        System.out.println(list);
```
json字符串数组转List ：

```java
String str="[{\"name\":\"张三\",\"age\":\"18\"}, {\"name\":\"李四\",\"age\":\"19\"}]";

//json字符串数组转List
ObjectMapper mapper = new ObjectMapper();
List<Student> list = Arrays.asList(mapper.readValue(str, Student[].class));
```

# 十一、json字符串格式化

有时候我们想把 json 字符串格式化输出，也就是该缩进的缩进该换行的换行，让它更美观的输出，可以像下面这样：

```java
        String str = "[{\"isSendPhone\":\"true\",\"id\":\"22258352\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"7\",\"count\":\"5\",\"serviceName\":\"流程助手\",\"startdate\":\"2022-02-09 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-08 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"xx有限责任公司\",\"serviceId\":\"21\",\"isSendeMail\":\"true\",\"email\":\"\"},{\"isSendPhone\":\"true\",\"eid\":\"7682130\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"15\",\"count\":\"50\",\"serviceName\":\"经理人自助服务\",\"startdate\":\"2022-02-17 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-16 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"xx科技股份有限公司\",\"serviceId\":\"2\",\"isSendeMail\":\"true\",\"email\":\"\"}]";
        
        ObjectMapper mapper = new ObjectMapper();
        String prettyStr = mapper.readTree(str).toPrettyString();
        System.out.println(prettyStr);

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

# 十二、其他

除此之外，jackson还提供了许多其他注解和流模式等，这里就不再举例了.

具体可参考官网：[jackson-databind](https://github.com/FasterXML/jackson-databind) 和 [Jackson JSON Tutorial](https://www.baeldung.com/jackson)

