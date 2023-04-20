[TOC]


# 前言
fastjson2 是 FASTJSON 项目的重要升级，目标是为下一个十年提供一个高性能的JSON库， fastjson2 性能相比原先旧的 fastjson 有了很大提升,并且 fastjson2 更安全，完全删除autoType白名单，提升了安全性。

[关于fastjson2升级指南](https://github.com/alibaba/fastjson2/wiki/fastjson_1_upgrade_cn)
[fastjson2 的github地址](https://github.com/alibaba/fastjson2)


# 一、导入fastjson2依赖
maven如下：
```xml
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
            <version>2.0.26</version>
        </dependency>
```
需要注意的一点是在使用 fastjson2 时导入的包名是 com.alibaba.fastjson2 ，就像下面这样：
```java
import com.alibaba.fastjson2.JSON;
import com.alibaba.fastjson2.JSONObject;
import com.alibaba.fastjson2.JSONArray;
```

# 二、json对象与json数组的创建
## json对象创建
```java
        JSONObject info = new JSONObject();
        info.put("name", "张三");
        info.put("age", "18");
        info.put("地理", "70");
        info.put("英语", "60");
```
## json数组创建
```java
		JSONObject info1 = new JSONObject();
        info1.put("name", "张三");
        info1.put("age", "18");
        JSONObject info2 = new JSONObject();
        info2.put("name", "李四");
        info2.put("age", "19");

        //把上面创建的两个json对象加入到json数组里
        JSONArray array = new JSONArray();
        array.add(info1);
        array.add(info2);
```

```java
        JSONArray array = new JSONArray();
        array.add("1班");
        array.add("2班");
        array.add("3班");
```

# 三、json对象取值与json数组遍历取值
## json对象取值
```java
        JSONArray array = new JSONArray();
        array.add("1班");
        array.add("2班");
        array.add("3班");
        JSONObject school = new JSONObject();
        school.put("schoolName", "第一中学");
        school.put("teacher", "刘梅");

        JSONObject info = new JSONObject();
        info.put("name", "张三");
        info.put("age", "18");
        info.put("gradle",array);
        info.put("schoolInfo",school);

        //从info中取值
        System.out.println(info.getString("name")); //张三
        System.out.println(info.getIntValue("age"));//18
        System.out.println(info.getJSONArray("gradle"));//["1班","2班","3班"]
        System.out.println(info.getJSONObject("schoolInfo"));//{"schoolName":"第一中学","teacher":"刘梅"}
```

## json数组遍历取值
```java
        JSONObject info1 = new JSONObject();
        info1.put("name", "张三");
        info1.put("age", "18");
        JSONObject info2 = new JSONObject();
        info2.put("name", "李四");
        info2.put("age", "19");

        JSONArray array = new JSONArray();
        array.add(info1);
        array.add(info2);
        //遍历获取json数组中对象的值
        for (int i = 0; i < array.size(); i++) {
            JSONObject json = array.getJSONObject(i);
            System.out.println(json.getString("name"));
            System.out.println(json.getString("age"));
        }
```

# 四、json对象与字符串的转换
## json对象与字符串的转换
```java
		JSONObject info = new JSONObject();
        info.put("name", "张三");
        info.put("age", "18");
        info.put("地理", "70");
        info.put("英语", "60");

        //JSON对象转字符串
        String str = JSON.toJSONString(info);
        //JSON字符串转JSON对象
        JSONObject json = JSONObject.parseObject(str);
```

## json字符串的字节数组转json对象
```java
        String str = "{\"name\":\"张三\",\"age\":\"18\",\"地理\":\"70\",\"英语\":\"60\"}";

        byte[] bytes = str.getBytes();
        JSONObject data = JSON.parseObject(bytes);

```

# 五、json数组与字符串的转换
```java
        String text = "[\"张三\",\"李四\",\"王五\"]";
        //json字符串转json数组
        JSONArray data = JSON.parseArray(text);
        //json数组转json字符串
        String str = JSONArray.toJSONString(data);
```

# 六、json字符串转java对象的转换

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
## json字符串转java对象的转换
```java
        Student student = new Student("张三", 18);

        //Student对象转JSON字符串
        String studentStr = JSON.toJSONString(student);
        //JSON字符串转Student对象
        Student data = JSON.parseObject(studentStr, Student.class);
```

## java对象转byte数组转换
```java
        Student student = new Student("张三", 18);

        //Student对象转字节数组
        byte[] text = JSON.toJSONBytes(student);
        //字节数组转Student对象
        Student data = JSON.parseObject(text, Student.class);

```


# 七、json字符串与Map转换
## json字符串转Map
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";

        //json字符串转Map
        Map<String, Object> map = JSONObject.parseObject(str, new TypeReference<Map<String, Object>>() {});
        System.out.println(map.get("gradle").toString());
        System.out.println(map.get("number").toString());
        System.out.println(map.get("people").toString());

```

## Map转json字符串
(注意：如果直接使用JSON.toJSONString(map)转换，因为"测试1"的值为null，转换的结果就会是 {"测试2":"hello"})
```java
		Map<String,String> map=new HashMap<>();
        map.put("测试1",null);
        map.put("测试2","hello");

		//{"测试2":"hello","测试1":null}
        String str = JSON.toJSONString(map, JSONWriter.Feature.WriteMapNullValue);

```
如果你使用的是老的fastjson1,可以像下面这样转换：
```java
		Map<String,String> map=new HashMap<>();
		map.put("测试1",null);
		map.put("测试2","hello");
		String str = JSON.toJSONString(map, SerializerFeature.WriteMapNullValue) ;
```

# 八、json数组转List
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";

        JSONObject strJson = JSONObject.parseObject(str);
        //获取people数组
        JSONArray people = strJson.getJSONArray("people");
        //json数组转List
        List<Map> list = people.toJavaList(Map.class);

```
如果你使用的是老的fastjson1,可以像下面这样转换：
```java
        String str="{\n" +
                "\"gradle\":\"高一\",\n" +
                "\"number\":\"2\",\n" +
                "\"people\":[{\"name\":\"张三\",\"age\":\"15\",\"phone\":\"123456\"},\n" +
                "         {\"name\":\"李四\",\"age\":\"16\",\"phone\":\"78945\"}]\n" +
                "}";

        JSONObject strJson=JSONObject.parseObject(str);//字符串转json对象
        String people = String.valueOf(strJson.getJSONArray("people"));
        List<Map<String,String>> list = (List<Map<String,String>>) JSONArray.parse(people);
```

# 九、json字符串格式化
有时候我们想把 json 字符串格式化输出，也就是该缩进的缩进该换行的换行，让它更美观的输出，可以像下面这样：

```java
        String str = "[{\"isSendPhone\":\"true\",\"id\":\"22258352\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"7\",\"count\":\"5\",\"serviceName\":\"流程助手\",\"startdate\":\"2022-02-09 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-08 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"顾问有限责任公司\",\"serviceId\":\"21\",\"isSendeMail\":\"true\",\"email\":\"\"},{\"isSendPhone\":\"true\",\"eid\":\"7682130\",\"phoneMessgge\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"readsendtime\":\"9\",\"countdown\":\"15\",\"count\":\"50\",\"serviceName\":\"经理人自助服务\",\"startdate\":\"2022-02-17 00:00:00.0\",\"insertTime\":\"2023-02-02 07:00:38.0\",\"enddate\":\"2023-02-16 23:59:59.0\",\"emailMessage\":\"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。\",\"phone\":\"\",\"companyname\":\"生物科技股份有限公司\",\"serviceId\":\"2\",\"isSendeMail\":\"true\",\"email\":\"\"}]";
        
        str =  str.trim();
        String formatStr = null;
        if (str.startsWith("[")) {
            JSONArray data = JSON.parseArray(str);
            formatStr = JSONArray.toJSONString(data, JSONWriter.Feature.PrettyFormat, JSONWriter.Feature.WriteMapNullValue, JSONWriter.Feature.WriteNullListAsEmpty);
        } else {
            JSONObject strJson = JSONObject.parseObject(str);
            formatStr = JSON.toJSONString(strJson, JSONWriter.Feature.PrettyFormat, JSONWriter.Feature.WriteMapNullValue, JSONWriter.Feature.WriteNullListAsEmpty);
        }
        System.out.println(formatStr);

```

输出结果：

```java
[
	{
		"isSendPhone":"true",
		"id":"22258352",
		"phoneMessgge":"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
		"readsendtime":"9",
		"countdown":"7",
		"count":"5",
		"serviceName":"流程助手",
		"startdate":"2022-02-09 00:00:00.0",
		"insertTime":"2023-02-02 07:00:38.0",
		"enddate":"2023-02-08 23:59:59.0",
		"emailMessage":"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
		"phone":"",
		"companyname":"XX顾问有限责任公司",
		"serviceId":"21",
		"isSendeMail":"true",
		"email":""
	},
	{
		"isSendPhone":"true",
		"eid":"7682130",
		"phoneMessgge":"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
		"readsendtime":"9",
		"countdown":"15",
		"count":"50",
		"serviceName":"经理人自助服务",
		"startdate":"2022-02-17 00:00:00.0",
		"insertTime":"2023-02-02 07:00:38.0",
		"enddate":"2023-02-16 23:59:59.0",
		"emailMessage":"为避免影响您的正常使用请及时续费，若已续费请忽略此信息。",
		"phone":"",
		"companyname":"XX科技股份有限公司",
		"serviceId":"2",
		"isSendeMail":"true",
		"email":""
	}
]
```

---

参考：
[fastjson2](https://github.com/alibaba/fastjson2)