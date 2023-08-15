@[TOC](目录)
# 一、什么是事件？

事件是发生在你正在编程的系统中的事情——当事件发生时，系统产生（或“触发”）某种信号，并提供一种机制，当事件发生时，可以自动采取某种行动（即运行一些代码）。事件是在浏览器窗口内触发的，并倾向于附加到驻留在其中的特定项目。这可能是一个单一的元素，一组元素，当前标签中加载的 HTML 文档，或整个浏览器窗口。有许多不同类型的事件可以发生。

例如：

- 用户选择、点击或将光标悬停在某一元素上。
- 用户在键盘中按下某个按键。
- 用户调整浏览器窗口的大小或者关闭浏览器窗口。
- 网页结束加载。
- 表单提交。
- 视频播放、暂停或结束。
- 发生错误。
- 你可以从这里（以及从 MDN 事件参考文档）中看出，有相当多的事件可以被触发。

为了对一个事件做出反应，你要给它附加一个事件处理器。这是一个代码块（通常是你作为程序员创建的一个 JavaScript 函数），在事件发生时运行。当这样一个代码块被定义为响应一个事件而运行时，我们说我们在注册一个事件处理器。注意，事件处理器有时候被叫做事件监听器——从我们的用意来看这两个名字是相同的，尽管严格地来说这块代码既监听也处理事件。监听器留意事件是否发生，处理器对事件发生做出回应。

备注： web 事件不是 JavaScript 语言的核心——它们被定义成内置于浏览器的 API。

## 示例：处理点击事件
在这个示例中，我们在页面中有一个 <button> 元素：

```html
<button>改变颜色</button>
```
然后我们有一些 JavaScript。我们将在下一节中更详细地讨论这个问题，但现在我们可以说：它为按钮的 "click" 事件添加了一个事件处理器，该处理器对该事件的反应是将页面背景设置为随机颜色：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

btn.addEventListener("click", () => {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
});
```



# 二、使用 addEventListener()
正如我们在上一个示例中所看到的，能够触发事件的对象有一个 addEventListener() 方法，这就是推荐的添加事件处理器的机制。

让我们仔细看一下上一个示例的代码：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

btn.addEventListener("click", () => {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
});
```
HTML <button> 元素将在用户点击按钮时触发一个事件。所以它定义了一个 addEventListener() 函数，我们在这里调用它。我们要传入两个参数：

字符串 "click"，表示我们要监听点击事件。按钮可以触发很多其他的事件，比如当用户将鼠标移到按钮上时（"mouseover" 事件），或者当用户按下一个键并且按钮被聚焦时（"keydown" 事件）。
当事件发生时所调用的函数。在我们的例子中，该函数生成一个随机的 RGB 颜色，并将页面 <body> 的 background-color 设置为该颜色。
把处理函数作为一个单独的命名函数也是可以的，像这样：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

function changeBackground() {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
}

btn.addEventListener("click", changeBackground);
```
## 监听其他事件
有许多不同的事件可以由一个按钮元素来触发。让我们来做个实验。

首先，在本地创建 random-color-addeventlistener.html 的副本，并在浏览器中打开。这只是我们已经玩过的简单的随机颜色示例的一个副本。现在试着依次将 click 改为以下不同的值，并观察示例中的结果：

focus 和 blur：当按钮被聚焦或失焦时，颜色会改变；尝试按下 tab 键来聚焦于按钮，再次按下该键来使按钮失去焦点。这些事件通常用于在聚焦时显示填入表单字段的信息，或者在表单字段填入不正确的值时显示错误信息。
dblclick：颜色只在按钮被双击时改变。
mouseover 和 mouseout：当鼠标指针在按钮上悬停，或指针移出按钮时，颜色分别会改变。
一些事件，如 click（点击事件），几乎对任何元素都可用。其他事件则更具体，只在某些情况下有用：例如，play 事件只对某些元素有效，如 <video> 元素。

## 移除监听器
如果你使用 addEventListener() 添加了一个事件处理器，你可以使用 removeEventListener() 方法再次删除它。例如，这将删除 changeBackground() 事件处理器：

```javascript
btn.removeEventListener("click", changeBackground);
```
事件处理器也可以通过传递 AbortSignal 到 addEventListener()，然后在拥有 AbortSignal 的控制器上调用abort()，从而删除事件处理器。例如，要添加一个可以使用 AbortSignal 来删除的事件处理器，可以这样做：

```javascript
const controller = new AbortController();

btn.addEventListener("click",
  () => {
    const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
    document.body.style.backgroundColor = rndCol;
  },
  { signal: controller.signal } // 向该处理器传递 AbortSignal
);
```
然后可以像这样删除上面代码创建的事件处理器：

```javascript
controller.abort(); // 移除任何/所有与该控制器相关的事件处理器
```
对于简单的小程序，清理旧的、未使用的事件处理器是没有必要的，但对于更大的、更复杂的程序，它可以提高效率。另外，删除事件处理器的能力允许你让同一个按钮在不同的情况下执行不同的动作：你所要做的就是添加或删除处理程序。

## 在单个事件上添加多个监听器
通过对 addEventListener() 的多次调用，每次提供不同的处理器，你可以为一个事件设置多个处理器：

```javascript
myElement.addEventListener("click", functionA);
myElement.addEventListener("click", functionB);
```
当点击按钮时，所有处理器函数都会运行。

## 了解更多
对于 addEventListener() 来说，还有很多强大的特性和选项。

这些有些超出了本文的范畴，如果你想了解更多，请参见 [addEventListener](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener) 和  [removeEventListener](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/removeEventListener)参考页面。

# 三、其他事件监听器机制
我们推荐你使用 addEventListener() 来注册事件处理器。这是最强大的方法，在更复杂的程序中，它的扩展性最好。然而，还有两种注册事件处理器的方法，你可能会看到：事件处理器属性和内联事件处理器。

事件处理器属性
可以触发事件的对象（如按钮）通常也有属性，其名称是 on，后面是事件的名称。例如，元素有一个属性 onclick。这被称为事件处理器属性。为了监听事件，你可以将处理函数分配给该属性。

例如，我们可以像这样重写随机颜色示例：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

btn.onclick = () => {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
};
```
你也可以将处理器属性分配给具名函数：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

function bgChange() {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
}

btn.onclick = bgChange;
```
对于事件处理器属性，你不能为一个事件添加一个以上的处理程序。例如，你可以在一个元素上多次调用 addEventListener('click', handler)，并在第二个参数中指定不同的函数：

```javascript
element.addEventListener("click", function1);
element.addEventListener("click", function2);
```
对于事件处理器属性来说，这是不可能的，因为任何后续尝试都会覆写较早设置的属性：

```javascript
element.onclick = function1;
element.onclick = function2;
```
## 内联事件处理器——不要使用
你可能也会在代码中看到这种形式：

```html
<button onclick="bgChange()">按下我</button>
```
```javascript
function bgChange() {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  document.body.style.backgroundColor = rndCol;
}
```
在 web 上发现的最早的注册事件处理器的方法涉及到事件处理器 HTML 属性（或内联事件处理器），如示例所示。属性值就是你想在事件发生时运行的 JavaScript 代码。上面的示例调用了同一页面上 <script> 元素内定义的一个函数，但你也可以直接在属性内插入 JavaScript，比如说：

```html
<button onclick="alert('你好，这是来自旧式事件处理器的一条消息');">
  按下我
</button>
```
你可以找到许多事件处理器属性的 HTML 属性的等价表示；但是，你不应该使用这些属性——它们被认为是不好的做法。如果你正在做一些非常快速的事情，使用事件处理器属性可能看起来很容易，但它们很快就会变得无法管理和低效。

首先，把你的 HTML 和你的 JavaScript 混在一起不是一个好主意，因为它变得难以阅读。把你的 JavaScript 分开是一个好的做法，如果它在一个单独的文件中，你可以把它应用到多个 HTML 文档中。

即使在单个文件中，内联事件处理器也不是个好主意。一个按钮是可以的，但如果你有 100 个按钮呢？你必须在文件中添加 100 个属性；这将很快变成一个维护的噩梦。通过使用 JavaScript，你可以很容易地为页面上的所有按钮添加一个事件处理函数，不管有多少个，使用这样的方法：

```javascript
const buttons = document.querySelectorAll("button");

for (const button of buttons) {
  button.addEventListener("click", bgChange);
}
```
最后，作为一项安全措施，许多常见的服务器配置将禁止内联 JavaScript。

你永远不应该使用 HTML 事件处理器属性——那些已经过时了，使用它们是不好的做法。

# 四、事件对象
有时候在事件处理函数内部，你可能会看到一个固定指定名称的参数，例如 event、evt 或 e。这被称为事件对象，它被自动传递给事件处理函数，以提供额外的功能和信息。例如，让我们稍稍重写一遍我们的随机颜色示例：

```javascript
const btn = document.querySelector("button");

function random(number) {
  return Math.floor(Math.random() * (number + 1));
}

function bgChange(e) {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  e.target.style.backgroundColor = rndCol;
  console.log(e);
}

btn.addEventListener("click", bgChange);
```

在这里，可以看到我们在函数中包括一个事件对象 e，并在函数中设置背景颜色样式在 e.target 上——它指的是按钮本身。事件对象 e 的 target 属性始终是事件刚刚发生的元素的引用。所以在这个例子中，我们在按钮上设置一个随机的背景颜色，而不是页面。

>备注： 参见事件委托来了解使用 event.target 的示例。

>备注： 可以使用任何喜欢的名称作为事件对象——只需要选择一个名称，然后可以在事件处理函数中引用它。开发人员最常使用 e/evt/event，因为它们很简单易记。保持一致总是好的——至少对自己。如果可能的话，对别人也是如此。

## 事件对象的额外属性
大多数事件对象都有一套标准的属性和方法，请参阅 Event 对象参考，以获得完整的列表。

一些事件对象添加了与该特定类型的事件相关的额外属性。例如，keydown 事件在用户按下一个键时发生。它的事件对象是 KeyboardEvent，它是一个专门的 Event 对象，有一个 key 属性，告诉你哪个键被按下：

```html
<input id="textBox" type="text" />
<div id="output"></div>
```

```javascript
const textBox = document.querySelector("#textBox");
const output = document.querySelector("#output");
textBox.addEventListener("keydown", (event) => {
  output.textContent = `You pressed "${event.key}".`;
});
```



# 五、阻止默认行为
有时，你会遇到一些情况，你希望事件不执行它的默认行为。最常见的例子是 Web 表单，例如自定义注册表单。当你填写详细信息并按提交按钮时，自然行为是将数据提交到服务器上的指定页面进行处理，并将浏览器重定向到某种“成功消息”页面（或相同的页面，如果另一个没有指定）。

当用户没有正确提交数据时，麻烦就来了——作为开发人员，你希望停止提交信息给服务器，并给他们一个错误提示，告诉他们什么做错了，以及需要做些什么来修正错误。一些浏览器支持自动的表单数据验证功能，但由于许多浏览器不支持，因此建议你不要依赖这些功能，并实现自己的验证检查。我们来看一个简单的例子。

首先，这里有一个简单的 HTML 表单，需要你填入名（first name）和姓（last name）：

```html
<form>
  <div>
    <label for="fname">First name: </label>
    <input id="fname" type="text" />
  </div>
  <div>
    <label for="lname">Last name: </label>
    <input id="lname" type="text" />
  </div>
  <div>
    <input id="submit" type="submit" />
  </div>
</form>
<p></p>
```
接下来是 JavaScript 代码——这里我们在 submit 事件（表单提交时触发提交事件）的处理程序中实现一个非常简单的检查，测试文本字段是否为空。如果是这样，我们就在事件对象上调用 preventDefault() 函数，停止表单提交，然后在我们的表单下面的段落中显示错误信息，告诉用户出了什么问题：

```javascript

const form = document.querySelector("form");
const fname = document.getElementById("fname");
const lname = document.getElementById("lname");
const para = document.querySelector("p");

form.addEventListener("submit", (e) => {
  if (fname.value === "" || lname.value === "") {
    e.preventDefault();
    para.textContent = "You need to fill in both names!";
  }
});
```
显然，这是一种非常弱的表单验证——例如，用户输入空格或数字提交表单，表单验证并不会阻止用户提交——但对于演示来说已经足够。输出如下：


# 六、事件冒泡
事件冒泡描述了浏览器如何处理针对嵌套元素的事件。

## 在父元素上设置监听器
考虑像这样的网页：

```html
<div id="container">
  <button>点我！</button>
</div>
<pre id="output"></pre>
```
这里有一个在其他元素（<div>）内部的按钮，可以说这里的 <div> 元素是其中包含元素的父元素。当我们在父元素附加单击事件处理器，并点击按钮时，会发生什么？

```javascript
const output = document.querySelector("#output");
function handleClick(e) {
  output.textContent += `你在 ${e.currentTarget.tagName} 元素上进行了点击\n`;
}

const container = document.querySelector("#container");
container.addEventListener("click", handleClick);
```


你会发现在用户单击按钮时，父元素上触发了单击事件：

>你在 DIV 元素上进行了点击

这是有道理的：按钮在 <div> 里面，所以当你点击按钮的时候，你也隐含地点击了它所在的元素。

## 冒泡示例
如果在按钮及其父元素上同时添加事件处理器，会发生什么？

```html
<body>
  <div id="container">
    <button>点我！</button>
  </div>
  <pre id="output"></pre>
</body>
```
让我们试着给按钮、它的父元素（<div>）以及包含它们的 <body> 元素添加点击事件处理器：

```javascript
const output = document.querySelector("#output");
function handleClick(e) {
  output.textContent += `你在 ${e.currentTarget.tagName} 元素上进行了点击\n`;
}

const container = document.querySelector("#container");
const button = document.querySelector("button");

document.body.addEventListener("click", handleClick);
container.addEventListener("click", handleClick);
button.addEventListener("click", handleClick);
```


你会发现在用户单击按钮时，所有三个元素都触发了单击事件：

>你在 BUTTON 元素上进行了点击
你在 DIV 元素上进行了点击
你在 BODY 元素上进行了点击

在这种情况下：

- 最先触发按钮上的单击事件
- 然后是按钮的父元素（<div> 元素）
- 然后是 <div> 的父元素（<body> 元素）

我们可以这样描述：事件从被点击的最里面的元素冒泡而出。

这种行为可能是有用的，也可能引起意想不到的问题。在接下来的章节中，我们将看到它引起的一个问题，并找到解决方案。

## 视频播放器示例
在这个示例中，我们的页面包含一个视频，最初它为隐藏状态；还有一个标记为“显示视频”的按钮。我们希望有如下交互：

当用户单击“显示视频”按钮时，显示包含视频的盒子，但不要开始播放视频。
当用户在视频上单击时，开始播放视频。
当用户单击盒子内视频以外的任何区域时，隐藏盒子。
HTML 代码看起来像这样：

```html
<button>显示视频</button>

<div class="hidden">
  <video>
    <source
      src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.webm"
      type="video/webm" />
    <p>
      你的浏览器不支持 HTML 视频，这里有视频的<a href="rabbit320.mp4"
        >替代链接</a
      >。
    </p>
  </video>
</div>
```
它包含：

- 一个 <button> 元素
- 一个 <div> 元素，最初其包含 class="hidden" 属性
- 一个嵌套在 <div> 元素中的 <video> 元素
我们使用 CSS 来隐藏具有 "hidden" 类的元素。

JavaScript 代码看起来像这样：

```javascript
const btn = document.querySelector("button");
const box = document.querySelector("div");
const video = document.querySelector("video");

btn.addEventListener("click", () => box.classList.remove("hidden"));
video.addEventListener("click", () => video.play());
box.addEventListener("click", () => box.classList.add("hidden"));
```

它添加了三个 'click' 事件处理器：

- 一个在 <button> 上，它显示了包含 <video> 的 <div>
- 一个在 <video> 上，用于开始播放视频
- 一个在 <div> 上，用于隐藏视频
让我们看看这个如何工作：


你应该看到，当你点击按钮时，盒子和它所包含的视频都显示出来。但当你点击视频时，视频开始播放，但盒子又被隐藏起来了！

视频在 <div> 内（是它的一部分），所以点击视频会同时运行两个事件处理器，导致这种行为。

## 使用 stopPropagation() 修复问题
正如我们在上一节所看到的，事件冒泡有时会产生问题，但有一种方法可以防止这些问题。Event 对象有一个可用的函数，叫做 stopPropagation()，当在一个事件处理器中调用时，可以防止事件向任何其他元素传递。

我们可以通过修改 JavaScript 代码来修复当前的问题：

```javascript
const btn = document.querySelector("button");
const box = document.querySelector("div");
const video = document.querySelector("video");

btn.addEventListener("click", () => box.classList.remove("hidden"));

video.addEventListener("click", (event) => {
  event.stopPropagation();
  video.play();
});

box.addEventListener("click", () => box.classList.add("hidden"));
```
我们在这里所做的是在 <video> 元素的 'click' 事件的处理器中对事件对象调用 stopPropagation()。这将阻止该事件向盒子内传递。现在试着点击按钮，然后再点击视频。

## 事件捕获
事件传播的另一种形式是事件捕获。这就像事件冒泡，但顺序是相反的：事件不是先在最内层的目标元素上发生，然后在连续较少的嵌套元素上发生，而是先在最小嵌套元素上发生，然后在连续更多的嵌套元素上发生，直到达到目标。

事件捕获默认是禁用的，你需要在 addEventListener() 的 capture 选项中启用它。

以下示例类似于之前看到的冒泡示例，除了使用了 capture 选项以外：

```html
<body>
  <div id="container">
    <button>点我！</button>
  </div>
  <pre id="output"></pre>
</body>
```
```javascript
const output = document.querySelector("#output");
function handleClick(e) {
  output.textContent += `你在 ${e.currentTarget.tagName} 元素上进行了点击\n`;
}

const container = document.querySelector("#container");
const button = document.querySelector("button");

document.body.addEventListener("click", handleClick, { capture: true });
container.addEventListener("click", handleClick, { capture: true });
button.addEventListener("click", handleClick);
```


在这种情况下，消息出现的顺序发生了颠倒：<body> 事件处理器首先触发，然后是 <div> 的，最后是 <button> 的：

>你在 BODY 元素上进行了点击
你在 DIV 元素上进行了点击
你在 BUTTON 元素上进行了点击

为什么要同时使用捕获和冒泡功能？在过去的坏日子里，当浏览器的交叉兼容性远不如现在时，Netscape 只使用事件捕捉，而 Internet Explorer 只使用事件冒泡。当 W3C 决定尝试将行为标准化并达成共识时，他们最终确定了这个包括这两种行为的系统，这也是现代浏览器所实现的。

默认情况下，几乎所有的事件处理程序都是在冒泡阶段注册的，这在大多数情况下更有意义。

# 七、事件委托
在上一节中，我们看了一个由事件冒泡引起的问题以及如何解决它。不过，事件冒泡并不只是令人讨厌：它可以非常有用。特别是，它可以实现事件委托。在这种做法中，当我们想在用户与大量的子元素中的任何一个互动时运行一些代码时，我们在它们的父元素上设置事件监听器，让发生在它们身上的事件冒泡到它们的父元素上，而不必在每个子元素上单独设置事件监听器。

让我们回到第一个例子，当用户点击一个按钮时，我们设置整个页面的背景颜色。假设取而代之的是，页面被分为 16 个区域，我们想在用户点击每个区域时将其设置为随机颜色。

这里是 HTML 代码：

```html
<div id="container">
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
  <div class="tile"></div>
</div>
```
我们有一些 CSS 代码，来设置每一个区域的尺寸和位置：

```html
.tile {
  height: 100px;
  width: 25%;
  float: left;
}
```
在 JavaScript 代码中，我们向每一个区域中添加单击事件处理器。但是，一个更简单、更有效的选择是在父节点上设置点击事件处理器，并依靠事件冒泡来确保用户点击每个区域时处理程序被执行：

```javascript
function random(number) {
  return Math.floor(Math.random() * number);
}

function bgChange() {
  const rndCol = `rgb(${random(255)}, ${random(255)}, ${random(255)})`;
  return rndCol;
}

const container = document.querySelector("#container");

container.addEventListener("click", (event) => {
  event.target.style.backgroundColor = bgChange();
});
```
>备注： 在这个例子中，我们使用 event.target 来获取事件的目标元素（也就是最里面的元素）。如果我们想访问处理这个事件的元素（在这个例子中是容器），我们可以使用 event.currentTarget。

# 八、综合示例
## 示例1
下面的代码点击按钮时按钮内的文字会改变，当再次点击时变回原样。
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <title>Events: Task 1</title>
    <style>
      p {
        color: purple;
        margin: 0.5em 0;
      }

      * {
        box-sizing: border-box;
      }

      button {
        display: block;
        margin: 20px 0 20px 20px;
      }
    </style>
    <link rel="stylesheet" href="../styles.css" />
  </head>

  <body>
    <section class="preview">
    </section>
    <button class="off">按钮</button>

  <script>
    const btn = document.querySelector('.off');
	btn.addEventListener("click",()=>btn.innerText = "按钮" == btn.innerText ? "这是改变文字后的按钮" : "按钮"); 
  </script>

  </body>
</html>
```

## 示例2
通过按下键盘上的 WSAD 四个键时，分别控制圆圈朝上下左右运动。该圆是使用函数drawCircle() 绘制的，该函数采用以下参数作为输入：

- x— 圆的 x 坐标。
- y— 圆的 y 坐标。
- size——圆的半径。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <title>Events: Task 2</title>
    <style>
      p {
        color: purple;
        margin: 0.5em 0;
      }

      * {
        box-sizing: border-box;
      }

      canvas {
        border: 1px solid black;
      }
    </style>
    <link rel="stylesheet" href="../styles.css" />
  </head>

  <body>
    <section class="preview">
    </section>
    <canvas width="480" height="320" tabindex="0">
    </canvas>
    
  <script>
    const canvas = document.querySelector('canvas');
    const ctx = canvas.getContext('2d');

    function drawCircle(x, y, size) {
      ctx.fillStyle = 'white';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.beginPath();
      ctx.fillStyle = 'black';
      ctx.arc(x, y, size, 0, 2 * Math.PI);
      ctx.fill();
    }

    let x = 50;
    let y = 50;
    const size = 30;

    drawCircle(x, y, size);

    // Add your code here
    canvas.addEventListener("keypress",(e)=>{
      if(e.keyCode==65||e.keyCode==97){//A
        x=x-1;
      }else if(e.keyCode==87||e.keyCode==119){//w
        y=y-1;
      }else if(e.keyCode==83||e.keyCode==115){//S
        y=y+1;
      }else if(e.keyCode==68||e.keyCode==100){//D
        x=x+1;
      }
      drawCircle(x, y, size);
    });
  </script>
  </body>
  
</html>
```

## 示例3
需要在<button> 的父元素 ( <div class="button-bar"> … </div>) 上设置事件侦听器，当通过单击任何按钮调用该事件侦听器时，会将 div 标签的背景设置为按钮data-color属性中的颜色。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <title>Events: Task 3</title>
    <style>
      p {
        color: purple;
        margin: 0.5em 0;
      }

      * {
        box-sizing: border-box;
      }

      button {
        display: block;
        margin: 20px 0 20px 20px;
      }

      .button-bar {
        padding: 20px 0;
      }
    </style>
    <link rel="stylesheet" href="../styles.css" />
  </head>

  <body>

    <section class="preview">
    </section>
    <div class="button-bar">
      <button data-color="red">Red</button>
      <button data-color="yellow">Yellow</button>
      <button data-color="green">Green</button>
      <button data-color="purple">Purple</button>
    </div>
    
  <script>

  const buttonBar = document.querySelector('.button-bar');

  buttonBar.addEventListener("click",(e)=>{
    buttonBar.style.backgroundColor = e.target.getAttribute("data-color");
  });

  </script>
  </body>
</html>
```


---
参考：

[事件介绍](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Building_blocks/Events)
[Test your skills: Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Test_your_skills:_Events)