@[TOC](目录)
# 一、字典（JSON 对象）与JSON字符串的转换

 ## 1.字典转JSON字符串

```python
import json

student = {"name": "张三", "age": 18, "gender": "男"}
# 将字典转成json字符串
jsonStr = json.dumps(student, ensure_ascii=False)
print(jsonStr)
```

## 2.JSON字符串转字典

```python
import json

studentStr = '{"name": "张三", "age": 18, "gender": "男"}'
# 将字符串转成字典
jsonDict = json.loads(studentStr)
```

# 二、列表（JSON 数组）与JSON字符串的转换
## 1.列表转JSON字符串

```python
import json

students = [{"name": "张三", "age": 18, "gender": "男"},{"name": "刘霜", "age": 19, "gender": "女"}]
# 将列表转成json字符串
jsonStr = json.dumps(students, ensure_ascii=False)
print(jsonStr)
```

## 2.JSON字符串转列表

```python
import json

studentsStr = '[{"name": "张三", "age": 18, "gender": "男"}, {"name": "刘霜", "age": 19, "gender": "女"}]'
# 将字符串转成列表
jsonList = json.loads(studentsStr)
```
# 三、格式化JSON字符串输出
对于JSON字符串，有时候我们不想那么紧凑的打印输出，而是想要根据层级缩进格式化的输出，可以在` json.dumps() `方法里加入` indent` 参数，比如 indent=2 每层就会缩进两个空格
```python
import json

students = [{"name": "张三", "age": 18, "gender": "男"}, {"name": "刘霜", "age": 19, "gender": "女"}]
# 将列表转成json字符串，并使用 indent=2 每层缩进两个空格
jsonStr = json.dumps(students, ensure_ascii=False, indent=2)
print(jsonStr)
```
输出效果：
```python
[
  {
    "name": "张三",
    "age": 18,
    "gender": "男"
  },
  {
    "name": "刘霜",
    "age": 19,
    "gender": "女"
  }
]
```

# 四、字典或列表读写到JSON文件


## 1.写入josn文件
以列表为例
```python
import json

students = [{"name": "张三", "age": 18, "gender": "男"}, {"name": "刘霜", "age": 19, "gender": "女"}]
with open('F:\pythonTest\myfile.json', 'w', encoding='utf8') as json_file:
    json.dump(students, json_file, ensure_ascii=False, indent=2)
```

## 2.读取josn文件
使用 json.load 读取json文件，如果文件中是json数组读取出来students是列表，如果是json对象读取出来students是字典。
```python
import json

#如果文件中是json数组读取出来students是列表，如果是json对象读取出来students是字典
with open("F:\pythonTest\myfile.json", "r", encoding='utf-8') as f:
    students = json.load(f)
```

# 五、总结

|      方法|  作用    |
| ---- | ---- |
|    json.dumps()   |   将python对象编码成Json字符串    |
|    json.loads()  |   将Json字符串解码成python对象   |
|    json.dump()  |    将python中的对象转化成json储存到文件中  |
|    json.load()  |    将文件中的json的格式转化成python对象提取出来  |

1.要将python中的对象（如字典、列表）转换成 json 字符串，使用 json 模块的 json.dumps( ) 方法。

2.要将python中的对象（如字典、列表）写入文件，使用 json 模块的 json.dump( ) 方法。

3.要将 JSON 字符串转成python中的对象（如字典、列表），使用 json 模块的 json.loads( ) 方法。

4.要将JSON文件读取为python中的对象（如字典、列表），使用 json 模块的 json.load( ) 方法。

5.ensure_ascii 与 indent 参数

- ensure_ascii=False ：如果字典或列表里有中文，不使用 `ensure_ascii=False` 参数的话 转化为JSON的字符串的时候会被转义为 \u5f20\u4e09 这样的字符 （即所有非 ASCII 字符都会被转义），所以上面的示例代码里都加了 `ensure_ascii=False` 来让中文不转义让它正常显示。如果你的字典或列表里没有中文没有非 ASCII 字符，在使用 json.dumps( ) 方法转成 JSON 字符串的时候可以不用加 `ensure_ascii=False`
- indent= n ：在字典或列表要转成 JSON 字符串转成字符串的时候，如果不想那么紧凑的打印JSON字符串，可以加入 `indent=n` 来格式化缩进 JSON  字符串，其中 n 是每层缩进的空格数。


---
参考：
[json — JSON encoder and decoder](https://docs.python.org/3.11/library/json.html#json.ensure_ascii)
[Python中json.load()和json.loads()之间的区别](https://www.pythonthree.com/difference-between-json-load-and-json-loads/)