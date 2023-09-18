[toc]

# 一、介绍
## 1.创建 XMLHttpRequest

```javascript
let xhr = new XMLHttpRequest();
```

## 2.初始化


```javascript
xhr.open(method, URL, [async, user, password])
```
使用 open 方法进行一些初始化配置，method 和 URL 是必传的，其余的是可选的。

**参数说明：**

method —— HTTP 方法。通常是 "GET" 或 "POST"，小写也可。
URL —— 要请求的 URL，通常是一个字符串，也可以是 URL 对象。
async —— 是否同步。如果不传默认为true，如果显式地设置为 false，那么请求将会以同步的方式处理。
user，password —— HTTP 基本身份验证（如果需要的话）的登录名和密码。


# 3.发送请求
```javascript
xhr.send([body])
```
使用 send 方法就会建立连接，发送请求。 

**参数说明：** 

body 是可选的，是 POST 请求方式的请求体。
即如果你上面的 xhr.open 里 method 是 使用的是 GET 请求 ，那么 body 参数是不需要的 ；
如果你上面的 xhr.open 里 method 是 使用的是 POST 请求，那么请求体就是 body 。


## 4.获取响应
下面这三个事件是最常用的：

- load —— 当请求完成（即使 HTTP 状态为 400 或 500 等），并且响应已完全下载。
- error —— 当无法发出请求，例如网络中断或者无效的 URL。
- progress —— 在下载响应期间定期触发，报告已经下载了多少。

 ```javascript
xhr.onload = function() { //请求正常发出时
  alert(`状态码: ${xhr.status} 结果： ${xhr.response}`);
};

xhr.onerror = function() { // 仅在根本无法发出请求时触发
  alert(`网络错误`);
};

xhr.onprogress = function(event) { // 定期触发，一般用于下载文件之类的返回下载进度
  // event.loaded —— 已经下载了多少字节
  // event.lengthComputable = true，当服务器发送了 Content-Length header 时
  // event.total —— 总字节数（如果 lengthComputable 为 true）
  alert(`总字节数： ${event.total} 已下载字节数： ${event.loaded} `);
};
 ```

## 5.响应类型
我们可以使用 xhr.responseType 属性来设置响应格式：

- ""（默认）—— 响应格式为字符串
- "text" —— 响应格式为字符串
- "arraybuffer" —— 响应格式为 ArrayBuffer（对于二进制数据，请参见 ArrayBuffer，二进制数组）
- "blob" —— 响应格式为 Blob（对于二进制数据，请参见 Blob），即文件数据
- "document" —— 响应格式为 XML document（可以使用 XPath 和其他 XML 方法）或 HTML document（基于接收数据的 MIME 类型）
- "json" —— 响应格式为 JSON（自动解析）


```javascript
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/example/json');

xhr.responseType = 'json';

xhr.send();

// 响应为 {"message": "Hello, world!"}
xhr.onload = function() {
  let responseObj = xhr.response;
  alert(responseObj.message); // Hello, world!
};

```


> 注意：
>
> 在旧的脚本中，你可能会看到 `xhr.responseText`，甚至会看到 `xhr.responseXML` 属性。
>
> 它们是由于历史原因而存在的，以获取字符串或 XML 文档。如今，我们应该在 `xhr.responseType` 中设置格式，然后就能获取如上所示的 `xhr.response` 了。




# 二、发送GET请求示例

```javascript
let xhr = new XMLHttpRequest();
xhr.open("GET", "http://localhost:8081/apiList/v1/worldAddressAnalysis?country=中国");
xhr.send();

//请求正常发出时
xhr.onload = function () { 
     alert(`状态码: ${xhr.status} 结果： ${xhr.response}`);
};

// 仅在根本无法发出请求时触发
xhr.onerror = function () { 
    alert(`网络错误`);
};
```

# 三、发送POST请求示例
```javascript
let xhr = new XMLHttpRequest();
xhr.open("POST", "http://localhost:8081/apiList/v1/calculator");
xhr.setRequestHeader('Content-Type', 'application/json; charset=utf-8');

//post 请求体参数
let param = {calculatorStr: "8*8-4"};
xhr.send(JSON.stringify(param));

//请求正常发出时
xhr.onload = function () { 
    alert(`状态码: ${xhr.status} 结果： ${xhr.response}`);
};

// 仅在根本无法发出请求时触发
xhr.onerror = function () { 
   alert(`网络错误`);
};
```

# 四、发送POST请求下载文件示例
```javascript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="btn">下载文件</button>

    <script>
        let btn = document.querySelector("#btn");
        btn.addEventListener("click", () => {
            let xhr = new XMLHttpRequest();
            xhr.open("POST", "http://localhost:8081/apiList/v1/downloadExcel");
            //设置请求头和响应类型
            xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8"); 
            xhr.responseType = 'blob';

            //设置请求体参数
            let param = { "fileName": "测试", "tableHeader": ["年级", "班级", "班主任", "男生人数", "女生人数", "语文平均分"] };
            xhr.send(JSON.stringify(param));

            //请求正常发出时
            xhr.onload = function () { 
                if (xhr.status === 200) {
                    // 获取文件blob数据并保存
                    saveAs(xhr.response, "download.xlsx");
                }
            };

            // 仅在根本无法发出请求时触发
            xhr.onerror = function () { 
                alert(`网络错误`);
            };

            // 定期触发 查看下载进度
            xhr.onprogress = function (event) { 
                alert(`总字节数： ${event.total} 已下载字节数： ${event.loaded} `);
            };
        });

        function saveAs(data, name) {
                var urlObject = window.URL || window.webkitURL || window;
                var export_blob = new Blob([data]);
                var save_link = document.createElementNS('http://www.w3.org/1999/xhtml', 'a')
                save_link.href = urlObject.createObjectURL(export_blob);
                save_link.download = name;
                save_link.click();
            }

    </script>
</body>

</html>
```

# 五、发送POST请求上传文件示例

假设我  springboot 上传文件的接口如下:

```java
    @ApiOperation("文件上传到jar包所在服务器")
    @PostMapping(value = "/v1/fileSelfUpload1", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public JSONObject fileSelfUpload1(@RequestParam String filePath, @RequestParam(value = "file", required = true) MultipartFile file) {
        return userConsumer.fileSelfUpload(filePath, file);
    }
```

使用 XMLHttpRequest 上传文件示例如下：

```javascript
<!DOCTYPE html>
<html lang="en-us">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Shopping list example</title>
    <style>
        li {
            margin-bottom: 10px;
        }

        li button {
            font-size: 8px;
            margin-left: 20px;
            color: #666;
        }
    </style>
</head>

<body>

    <div class="container">
        <input type="file" name="doc" id="doc">
        <button type="submit" id="submit">提交</button>
    </div>

    <script>
        var doc = document.querySelector('#doc')
        var subbtn = document.querySelector('#submit')

        subbtn.addEventListener('click', function (e) {
            // 获取上传的文件,数组
            let filedata = doc.files

            //先判断是否已经上传文件
            if (filedata.length <= 0) {
                return alert('请上传文件')
            }

            //通过FormData可以获取表单数据，也可以通过append添加数据，最后把FormData实例对象直接  上传发送请求
            let fd = new FormData()
            fd.append('file', filedata[0])
            fd.append('filePath', '/user/save/')

            const xhr = new XMLHttpRequest()
            xhr.open('POST', 'http://localhost:8081/apiList/v1/fileSelfUpload1')
            xhr.send(fd)

            xhr.onreadystatechange = function () {
                if (xhr.readyState === 4 && xhr.status === 200) {
                    console.log('上传成功');
                }
            }

        })
    </script>
</body>

</html>
```

----

参考：

[XMLHttpRequest](https://zh.javascript.info/xmlhttprequest)

[XMLHttpRequest api](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
