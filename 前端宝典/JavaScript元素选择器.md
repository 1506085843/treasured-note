[TOC]



# 一、getElementsByTagName

## 1.说明

getElementsByTagName 俗称标签选择器，可以根据标签名查找匹配到页面的元素对象，返回为一个数组。

## 2.用法示例

获取所有 p 标签的文本。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
</head>

<body>
    <button onclick="alertInfo();"> 获取p标签文本 </button>
        <p>这是第一段文本aaaa</p>
        <p>这是第二段文本bbbb</p>
        <p>这是第三段文本cccc</p>
        <p>这是第四段文本dddd</p>
</body>
<script>
    function alertInfo() {
        const matches = document.getElementsByTagName("p");
        let pText = "";
        for (const el of matches) {
            pText = pText + el.innerText + "\n";
        }
        alert("p标签文本是：\n" + pText);
    }
</script>

</html>
```

# 二、getElementsByName

## 1.说明

getElementsByName 俗称name选择器，可以根据name属性的值查找匹配到页面的元素对象，返回为一个数组。

## 2.用法示例

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
</head>

<body>
    <button onclick="alertInfo();"> 获取name为test的文本 </button>
        <p>这是第一段文本aaaa</p>
        <p name="test">这是第二段文本bbbb</p>
        <p name="test">这是第二段文本cccc</p>

</body>
<script>
    function alertInfo() {
        const matches = document.getElementsByName("test");
        let test = "";
        for (const el of matches) {
            test = test + el.innerText + "\n";
        }
        alert( test);
    }
</script>

</html>
```



# 三、getElementById

## 1.说明
getElementById 俗称 id 选择器 ， **`getElementById(id)`**  方法查找并返回一个与页面中 id 相匹配的元素对象。 

一般来说 id  在页面中应该是唯一的，因此该方法是快速访问特定元素的方法，如果页面中的两个或多个元素具有相同的 id，则此方法返回找到的第一个元素。

## 2.用法示例
两个 p 标签含有一样的 id ，getElementById 方法只会返回找到的第一个元素，所以点击按钮后只有第一段文字的颜色会改变。
```html
<html lang="en">
<head>
    <title>getElementById 示例</title>
</head>
<body>
    <p id="para">这是第一段测试文字</p>
    <p id="para">这是第二段测试文字</p>
    <button onclick="changeColor('blue');">蓝色</button>
    <button onclick="changeColor('red');">红色</button>
</body>
<script>
    function changeColor(newColor) {
        const elem = document.getElementById("para");
        elem.style.color = newColor;
    }
</script>
</html>
```


# 四、getElementsByClassName
## 1.说明
getElementsByClassName 俗称 class 选择器， getElementsByClassName(class) 方法查找页面上所有类名为 class 的元素对象，返回为一个数组。（方法里面要查找的多个类名之间用空格分隔。）

## 2.用法示例
(1)获取所有 class="test" 的元素
```html
document.getElementsByClassName("test");
```
(2)获取同时具有 'red' 和 'test' 类的所有元素
```html
document.getElementsByClassName("red test");
```
(3)获取 id="main" 的元素且内部具有 class="test" 的所有元素
```html
document.getElementById("main").getElementsByClassName("test");
```
(4) 获取第一个 class="test" 的元素（如果没有匹配的元素返回undefined）
```html
document.getElementsByClassName("test")[0];
```
(5)简单示例
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
</head>
<body>
    <button onclick="changeColor();"> 改变第一段文字颜色 </button>
    <div class="info"> 这是第一段文字</div>
    <div class="info"> 这是第二段文字</div>
</body>
<script>
    function changeColor() {
        const infos = document.getElementsByClassName("info");
        infos[0].style.color = "red";
    }
</script>
</html>
```

# 五、querySelector

## 1.说明

querySelector 是元素选择器，可用于 id 和 class 选择，也就是上面 getElementById 和 getElementsByClassName 能做到的 querySelector 也能做到，并且 querySelector 还能用于其他复杂的元素选择场景，最后返回查找的元素。

## 2.用法示例

(1)  取得DOM中第一个 id= “box” 的元素

```html
document.querySelector("#box") 
```

(2)  取得DOM中第一个 class= “box” 的元素

```html
document.querySelector(".box") 
```

（3）选择器中逗号分割表示或者

querySelector 里可用逗号分割来表示或者的意思，下面的示例姓名输入框在年龄输入框前所以获取到的是姓名，若年龄输入框在前就会获取到年龄。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset = "utf-8">
</head>

<body>
    <button onclick = "alertInfo();"> 获取姓名 </button>
    <div> 
        姓名： <input name = "login" class="name" value = "张三" />
        年龄： <input name = "logina" class="age"value = "18" />
    </div>
</body>
<script>
    function alertInfo() {
        //获取class="name"的input元素，或者class="age"的input元素
        const el = document.querySelector("input.name,input.age");
        alert("姓名或年龄是：" + el.value);
    }
</script>

</html>
```

（4）复杂场景的选择器使用

查找页面  div 标签里 class="user-panel main"  的元素中 第一个 name = "login" 的 input 元素。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset = "utf-8">
    <style>
        .user-panel{
            margin-top: 25px;
        }
        .main{
            margin-left: 10px;
        }
    </style>
</head>

<body>
    <button onclick = "alertInfo();"> 获取姓名 </button>
    <div class = "user-panel main"> 
        姓名： <input name = "login" value = "张三" />
    </div>
    <div class = "user-panel main"> 
        年龄： <input name = "logina" value = "18" />
    </div>
</body>
<script>
    function alertInfo() {
        const el = document.querySelector("div.user-panel.main input[name='login']");
        alert("输入的姓名是：" + el.value);
    }
</script>

</html>
```

# 六、querySelectorAll

## 1.说明

querySelectorAll 选择器和 querySelector 选择器相似，只不过 querySelector 返回的是匹配的一个元素，querySelectorAll 返回的是匹配的多个元素，即数组类型。

## 2.用法示例

（1）获取所有p标签元素

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset = "utf-8">
</head>

<body>
    <button onclick = "alertInfo();"> 获取姓名 </button>
    <p>这是第一段文本aaaa</p>
    <p>这是第二段文本bbbb</p>
</body>
<script>
    function alertInfo() {
        //获取所有p标签元素
        const firstP = document.querySelectorAll("p")
        //选取第一个p标签里的内容输出
        alert( firstP[0].innerText);
    }
</script>

</html>
```

(2)获取元素下的所有子元素

获取 id="two" 的 div 元素，然后获取其中 class="highlighted" 的 div 元素下是所有 p 标签。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <style>
        .highlighted {
            color: aqua;
        }
    </style>
</head>

<body>
    <button onclick="alertInfo();"> 获取姓名 </button>
    <div id="first">
        <p>这是第一段文本aaaa</p>
        <p>这是第二段文本bbbb</p>
    </div>
    <div id="two">
        <p>这是第三段文本cccc</p>
        <p>这是第四段文本dddd</p>
        <div class="highlighted">
            <p>这是第五段文本eeee</p>
            <p>这是第六段文本ffff</p>
        </div>
    </div>

</body>
<script>
    function alertInfo() {
        const container = document.querySelector("div#two");
        const matches = container.querySelectorAll("div.highlighted > p");
        let highlighText = "";
        for (const el of matches) {
            highlighText=highlighText+el.innerText+"\n";
        }
        alert("高亮文本是：\n"+highlighText);
    }
</script>

</html>
```

# 七、综合示例

1.点击按钮将会弹出弹窗。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
</head>

<body>
    <button> 点击 </button>
</body>
<script>
    let btn = document.querySelector("button");
    btn.addEventListener("click", alterFun);
    function alterFun(){
        alert("你好!");
    };
 </script>
</html>
```

2.点击按钮将会在页面上新增一个百度超链接并设置一些简单的样式。

```html

<head>
    <meta charset="utf-8">
</head>

<body>
    <button> 点击 </button>
</body>
<script>
    let btn = document.querySelector("button");
    btn.addEventListener("click", addA);
    function addA() {
        let newA = document.createElement("a");
        newA.href = "https://www.baidu.com";
        newA.innerText = "百度";
        newA.style.color = "red";
        newA.style.marginLeft = "100px";
        newA.target = "_blank";
        document.body.appendChild(newA);
    }
</script>

</html>
```

3.点击改变按钮将会 改变div里的两段文字并在其中增加a标签，点击还原第一段文字按钮将还原第一段文字。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <style>
        .b2 {
            color: coral;
        }
        .info{
            color: aqua;
        }
    </style>
</head>

<body>
    <button id="b1"> 改变 </button>
    <button class="b2"> 还原第一段文字 </button>
    <div class="info"> 这是第一段文字</div>
    <div class="info"> 这是第二段文字</div>
</body>
<script>
    //获取所有id为“b1”的元素，并将它们存储在名为“elements”的变量中
    let btn1 = document.getElementById("b1");
    btn1.addEventListener("click", change);
    function change() {
        //遍历所有class为“info”的元素,修改其中的文本和字体颜色，并在其中加入一个a标签链接
        let elements = document.getElementsByClassName("info");
        for (let element of elements) {
            //改变标签里的文字和颜色
            element.innerText = "这是一段新的文字";
            element.style.color = "red";
            //添加a标签
            let newA = document.createElement("a");
            newA.href = "https://www.baidu.com";
            newA.innerText = "百度";
            newA.style.marginLeft = "100px";
            newA.target = "_blank";
            element.appendChild(newA);
        }
    }

    //还原第一段改变的文字和颜色
    let btn2 = document.getElementsByClassName("b2");
    btn2[0].addEventListener("click",reset);
    function reset() {
        let infoDiv = document.getElementsByClassName("info");
        infoDiv[0].innerText = "这是一段文字";
        infoDiv[0].style.color = "aqua";
    }

</script>

</html>
```

-----

参考：

[Document: getElementById() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById)

[Document: getElementsByName() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByName)

[Document: querySelector() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector)

[Document: querySelectorAll() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll)