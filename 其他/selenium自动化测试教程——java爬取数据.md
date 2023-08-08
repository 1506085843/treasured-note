@[TOC](目录)

# 一、介绍

[selenium](https://www.selenium.dev/)  是一个用于自动化测试 Web 应用的工具集 ，它可以模拟用户自动去浏览器网页上进行点击、输入、选择下拉值复选框、鼠标移动、任意 JavaScript 执行等等操作。


[selenium](https://www.selenium.dev/) 有三个产品：

- [Selenium WebDriver](https://www.selenium.dev/documentation/webdriver/)：基于浏览器的回归自动化套件和测试，你可以使用 Java、Python、JavaScript、Ruby、JavaScript、C# 这些语言中的一种来编写代码，Selenium WebDriver 会根据代码去打开浏览器自动去网页上进行操作和测试。
- [Selenium IDE](https://www.selenium.dev/selenium-ide/)：selenium 开发的浏览器里的一款插件，它是界面化的操作，不用编写代码。如果你使用的谷歌浏览器你可以去谷歌插件应用商店搜索 Selenium IDE 进行安装和使用，它就像一个记录器一样，你可以在网页上进行点击、输入、跳转等操作，它会把你所有操作都记录下来，当你点击运行的时候它会把记录的操作自动化的操作一遍。Selenium IDE 的使用可以参考：[Selenium IDE教程](https://www.javatpoint.com/selenium-ide-features)
- [Selenium Grid](https://www.selenium.dev/documentation/grid/)：通过在多台机器上分布式运行测试，可以从一个中心点管理多个环境，从而轻松针对大量浏览器/操作系统进行组合运行测试。

本文讲的是使用 Selenium WebDriver 通过 java 代码来自动对网页进行操作，推荐使用 Chrome 浏览器来操作。


# 二、下载浏览器驱动

## 1.获取要下载的驱动版本号
![请添加图片描述](https://img-blog.csdnimg.cn/a950112575c046f7a7521a085793b6bb.jpeg)


Chrome 浏览器里查看你当前的版本，我这里是114.0.5735.134 ，丢弃最后一位数，得到 114.0.5735 ，然后拼接到`https://chromedriver.storage.googleapis.com/LATEST_RELEASE_` 会得到一个链接，我得到的链接如下：

```java
https://chromedriver.storage.googleapis.com/LATEST_RELEASE_114.0.5735
```

浏览器访问该链接会得到一个版本号

![请添加图片描述](https://img-blog.csdnimg.cn/d4f70d07df1d42c49b7a07390e3b22ae.jpeg)


我这里得到的是 114.0.5735.90 ，说明我应该要下载 114.0.5735.90 的驱动。

（关于版本选择可参考: [Version Selection](https://sites.google.com/chromium.org/driver/downloads/version-selection)）

## 2.下载驱动

根据上面得到的版本号，去 [ChromeDriver 下载](https://sites.google.com/chromium.org/driver/downloads) 页面选择对应的谷歌浏览器版本号的驱动。

![请添加图片描述](https://img-blog.csdnimg.cn/719b692f0d5041119bbe27d17dc86003.jpeg)


然后根据你电脑的操作系统下载对应驱动并解压（我这里下载的是 windos 版的），解压后得到 chromedriver.exe。

![请添加图片描述](https://img-blog.csdnimg.cn/74713a2bd3284f27b726a7bf8c397c05.jpeg)



# 三、Maven如下

```java
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.10.0</version>
        </dependency>
```

# 四、简单使用
下面的例子将会自动打开一个新的浏览器窗口，然后自动打开百度并自动搜索 "csdn 西凉的悲伤"，然后自动点击打开第一个搜索结果即我的博客主页，然后把抓取博客主页的文章目录和链接。

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;
import java.util.List;

import static org.openqa.selenium.support.ui.ExpectedConditions.numberOfWindowsToBe;
import static org.openqa.selenium.support.ui.ExpectedConditions.titleIs;


public class MainServer {
    public static void main(String[] args) {
        //加载 chromedriver 驱动
        System.setProperty("webdriver.chrome.driver", "D:\\Program\\chromedriver\\chromedriver.exe");
        //打开一个浏览器窗口
        WebDriver driver = new ChromeDriver();
        //打开百度链接
        driver.navigate().to("http://www.baidu.com/");
        //在搜索文本框输入"csdn 西凉的悲伤"
        driver.findElement(By.id("kw")).sendKeys("csdn 西凉的悲伤");
        //点击搜索按钮
        driver.findElement(By.id("su")).click();


        //存储当前原始窗口或页签的ID
        String originalWindow = driver.getWindowHandle();
        //获取当前打开的窗口或页签数
        int windosSize = driver.getWindowHandles().size();

        //等到百度搜索结果页面元素加载完（这里最多等5秒）
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5));
        //点击第一条搜索结果，会打开新页签，也就是第2个页签
        driver.findElement(By.xpath("//*[@id='content_left']/div[@id='1']/div[@class='c-container']/div/h3/a")).click();


        WebDriverWait wait = new WebDriverWait(driver, Duration.ofMillis(10));
        //等待第2个新窗口或新页签打开
        wait.until(numberOfWindowsToBe(2));
        //循环指导找到新窗口或页签的句柄
        for (String windowHandle : driver.getWindowHandles()) {
            if(!originalWindow.contentEquals(windowHandle)) {
                //driver切换为新窗口或新页签的
                driver.switchTo().window(windowHandle);
                break;
            }
        }
        //等待新窗口或新页签的内容加载
        wait.until(titleIs("西凉的悲伤的博客_CSDN博客-java,工具,其他领域博主"));


        //读取当前页面标题
        System.out.println("当前网址的标题："+driver.getTitle());
        //从地址栏中读取当前 URL
        System.out.println("当前网址的链接："+driver.getCurrentUrl());
        System.out.println();

		//获取多个 h4 标签元素
        List<WebElement> articleTitles = driver.findElements(By.xpath("//*[@class='blog-list-box-top']/h4"));
        //获取多个 a 标签元素
        List<WebElement> articleUrls = driver.findElements(By.xpath("//*[@class='blog-list-box']/a"));
        for (int i = 0; i < articleTitles.size(); i++) {
        	//获取 h4 标签中的显示文本
            String articleTitle = articleTitles .get(i).getText();
            //获取 a 标签里的 href 属性的值
            String articleUrl = articleUrls.get(i).getAttribute("href");
            System.out.println("文章标题："+articleTitle+" 链接："+articleUrl);
        }
    }
}
```

效果如下：
![请添加图片描述](https://img-blog.csdnimg.cn/ce8da14dcebc441b8101c5f49758af3e.jpeg)


1.上面的代码使用了System.setProperty 来加载驱动，当然，你也可以把它配置到环境变量里就不用从代码里加载驱动了。你可以参考这篇文章来配置驱动的环境变量：[selenium配置使用chromedriver](https://blog.csdn.net/kun_csdn/article/details/124267821)。

2.上面的代码使用了 implicitlyWait 方法来显式等待页面加载完，然后再去查找第一条搜索结果并点击，如果不等页面加载完就查找会找不到并报错。除了显式等待，还有隐式等待、流利等待，你可参考官网的说明：[selenium Waits](https://www.selenium.dev/documentation/webdriver/waits/) 。或者你也可以使用 Thread.sleep(2000L); 来让线程等待 2 秒。

3.如果想不打开浏览器即不打开浏览器 GUI，只让程序在后台跑把数据加载到内存在内存操作输出结果（即无头浏览器），可以把上面第18行代码替换为如下：

```java
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless"); //无浏览器模式
        options.addArguments("--disable-gpu"); // 谷歌文档提到需要加上这个属性来规避bug
        WebDriver driver = new ChromeDriver(options);
```

# 五、定位器

网页上有很多元素按钮和文本等东西，比如：如果你想自动点击登录按钮，需要先找到登录按钮；如果你想点击某个链接需要先找到该链接才能点击。

## 1.定位器

帮我们找网页上的元素的这个东西就叫做定位器，上面的示例里使用了 xpath 定位器，不止 xpath 定位器，selenium 还为我们提供了其他定位器方便我们来查找元素，比如：通过 class 名称、通过 id 名称、通过网页上显示的文本、通过 html 标签的层级等。

| 定位器 |    描述  |
| ------ | ---- |
| class name | 根据class 的值来搜索匹配元素 |
| css selector | 根据 css 的值来搜索匹配元素 |
| id |   根据 id 属性的值来搜索匹配元素   |
| name | 根据 name 属性的值来搜索匹配元素 |
| link text | 根据链接显示的全部文本搜索匹配元素 |
| partial link text | 根据链接显示的部分文本搜索匹配元素 |
| tag name | 根据html标签名搜索匹配元素 |
| xpath | 根据元素的层级位置搜索匹配元素 |

## 2.说明
以以下 html 为例，对上面的定位器进行说明。

```html
<html>
<body>
<style>
.information {
  background-color: white;
  color: black;
  padding: 10px;
}
</style>
<h2>Contact Selenium</h2>

<form action="/action_page.php">
  <input type="radio" name="gender" value="m" />Male &nbsp;
  <input type="radio" name="gender" value="f" />Female <br>
  <br>
  <label for="fname">First name:</label><br>
  <input class="information" type="text" id="fname" name="fname" value="Jane"><br><br>
  <label for="lname">Last name:</label><br>
  <input class="information" type="text" id="lname" name="lname" value="Doe"><br><br>
  <label for="newsletter">Newsletter:</label>
  <input type="checkbox" name="newsletter" value="1" /><br><br>
  <input type="submit" value="Submit">
</form> 

<p>To know more about Selenium, visit the official page 
<a href ="www.selenium.dev">Selenium Official Page</a> 
</p>

</body>
</html>

```

### (1) class name 定位器
HTML 页面 Web 元素可以具有class属性，我们可以使用 Selenium 中可用的类名定位器来识别这些元素。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.className("information"));
```
### (2) css selector 定位器
CSS 是用于设置 HTML 页面样式的语言。我们可以使用 css 选择器定位器策略来识别页面上的元素。如果元素有一个 id，我们创建定位器为 css = #id。否则我们遵循的格式是 css =[attribute=value] 。下面使用 css 为名字文本框创建定位器。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.cssSelector("#fname"));
```
### (3) id 定位器
我们可以使用网页中元素可用的 ID 属性来定位它。通常，ID 属性对于网页上的元素应该是唯一的。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.id("lname"));
```
### (4) name 定位器
我们可以使用网页中元素可用的 NAME 属性来定位它。通常 NAME 属性对于网页上的元素应该是唯一的。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.name("newsletter"));
```
### (5) link text 定位器
如果我们要定位的元素是一个链接，我们可以使用链接文本定位器在网页上识别它。链接文本是链接显示的文本。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.linkText("Selenium Official Page"));
```
### (6) partial link text 定位器
如果我们要定位的元素是一个链接，我们可以使用部分链接文本定位器在网页上识别它。链接文本是链接显示的文本。我们可以将部分文本作为值传递。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.partialLinkText("Official Page"));
```
### (7) tag 定位器
我们可以使用 HTML TAG 本身作为定位器来识别页面上的 Web 元素。使用tag 定位器来定位“a”标签。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.tagName("a"));
```
### (8) xpath 定位器
一个HTML文档可以看作是一个XML文档，然后我们就可以使用xpath来遍历到达感兴趣元素的路径来定位元素。XPath 可以是绝对 xpath，它是从文档的根目录创建的。示例 - /html/form/input[1]。这将返回男性单​​选按钮。或者 xpath 可能是相对的。示例： //输入[@name='fname']。这将返回名字文本框。让我们使用 xpath 为女性单选按钮创建定位器。
```java
    WebDriver driver = new ChromeDriver();
	driver.findElement(By.xpath("//input[@value='f']"));
```

关于 xpath 定位器你可以参考文章：
 [Selenium 中的 XPath ](https://blog.csdn.net/vvoennvv/article/details/127661533)
  [selenium 定位元素](https://blog.csdn.net/u014651560/article/details/130101433)
  [XPath in Selenium: How to Find & Write](https://www.guru99.com/xpath-selenium.html)
[How to use XPath in Selenium](https://www.browserstack.com/guide/xpath-in-selenium)

### (9) Selenium IDE 插件辅助定位元素
如果你对 html 不熟悉或者用定位器来找网页元素不方便，你可以先使用本文介绍部分提到的 Selenium IDE 插件操作一遍，然后选择保存项目，它会把所有操作保存成一个 `.side` 后缀格式的文件，文件里面有每一步操作的描述，
并且每一步操作里面的 targets 生成了多种定位器，你代码里直接使用 targets 里的一种定位器去直接写就好了，就不用自己去分析网页编写定位器定位元素了。

![请添加图片描述](https://img-blog.csdnimg.cn/9d320e6112914e5bba8d72831ff4fe90.jpeg)

![请添加图片描述](https://img-blog.csdnimg.cn/e74061e029f446448d55bfa5ae2ca987.jpeg)
### (10) chrome浏览器开发控制台定位元素后复制Xpath
还有一种简单的方式。chrome 浏览器网页上右键检查（或 F12），点击左上角的箭头，然后点击网页上的元素进行定位，再定位后的代码上右键复制里选择复制 XPath , 你就可以很方便的在代码里使用 xpath 定位器了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/abec74b9d4bd4bbf9aa44f9fa805b9b9.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0adbd2d7c5a7435096b43b742a2c3cb0.png)


# 六、常见操作

## 1.打开一个新的浏览器页签
```java
//打开一个新的浏览器页签
driver.switchTo().newWindow(WindowType.TAB);
```
## 2.打开网址链接
```java
//方便的方式
driver.get("http://www.baidu.com");
//或者下面的方式，是一样的效果
driver.navigate().to("http://www.baidu.com");
```

## 3.获取当前网页的标题和链接
```java
//读取当前页面标题
driver.getTitle();
//从地址栏中读取当前 URL
driver.getCurrentUrl();
```


## 4.浏览器前进、后退、刷新、关闭
```java
//浏览器的后退
driver.navigate().back();
//浏览器的前进
driver.navigate().forward();
//浏览器的刷新
driver.navigate().refresh();
//关闭浏览器
driver.quit();
```
## 5.弹窗的警告、确认

（1）获取警告弹窗的文本并点击确认
```java
//使用link text定位器找到页面链接，并点击它来出发弹窗 
driver.findElement(By.linkText("See an example alert")).click();
//等弹窗显示并获取弹窗对象
Alert alert = wait.until(ExpectedConditions.alertIsPresent());
//获取弹窗的文本内容
String text = alert.getText();
//点击弹窗的确认按钮
alert.accept();
```

（2）确认弹窗类似于警告弹窗，除了用户还可以选择取消消息。
此示例还展示了另一种获取弹窗对象的方法：
```java
//使用link text定位器找到链接，并点击它来出发弹窗 
driver.findElement(By.linkText("See a sample confirm")).click();
//等弹窗显示
wait.until(ExpectedConditions.alertIsPresent());
//获取弹窗对象
Alert alert = driver.switchTo().alert();
//获取弹窗的文本内容
String text = alert.getText();
//点击弹窗的取消按钮
alert.dismiss();
```
（3）可输入的弹窗
提示类似于确认弹窗，可输入的弹窗还可以输入一些文本信息，与使用表单元素类似。
```java
//使用link text定位器找到链接，并点击它来出发弹窗 
driver.findElement(By.linkText("See a sample prompt")).click();
//等弹窗显示并获取弹窗对象
Alert alert = wait.until(ExpectedConditions.alertIsPresent());
//在弹窗的输入框输入“你好啊”
alert.sendKeys("你好啊");
//按确定按钮
alert.accept();
```

## 6.判断页面的元素是否存在

```java
int headImage = driver.findElements(By.xpath("//*[@class='user_avatar']")).size();
if (headImage == 0) {
 	System.out.println("页面上 class 为 user_avatar 的 html 元素不存在");
}
```

## 7.在已打开的浏览器打开网址链接
默认的如果你使用 selenium 进行操作，selenium  会打开一个新的浏览器窗口然后再打开链接，并且这个新打开的浏览器窗口是没有插件 的（即使你浏览器里安装了插件）。那如何在已经打开的浏览器里打开链接呢？

（1）先关闭你的浏览器

（2）找到你浏览器的启动目录
选择浏览器图标，右键选择属性，点击打开文件所在的位置，然后复制该文件夹路径。

（3）新建个叫 chromeStart 的 txt 文件，内容如下，把下面第一行的路径替换成你上面复制的文件夹路径，然后保存
```bash
cd C:\Program Files\Google\Chrome\Application
start chrome.exe --flag-switches-begin --flag-switches-end --remote-debugging-port=9222
```

（4）把上面的 chromeStart.txt 文件后缀改为 bat  然后双击 chromeStart.bat 会打开浏览器

（5） selenium 代码要如下

```java
        //加载 chromedriver 驱动
        System.setProperty("webdriver.chrome.driver", "D:\\Program\\chromedriver\\chromedriver.exe");

        //使用已经打开的浏览器窗口
        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("debuggerAddress", "127.0.0.1:9222");
        WebDriver driver = new ChromeDriver(options);
        //打开一个新的页签
        driver.switchTo().newWindow(WindowType.TAB);
        //打开链接
        driver.navigate().to("https://www.baidu.com");
```

# 七、使用 cookie
## 1.添加cookie
```java
public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        try {
        	//打开网址
            driver.get("http://www.example.com");
            //添加cookie到当前浏览器网址的上下文中
            driver.manage().addCookie(new Cookie("key", "value"));
        } finally {
        	//关闭浏览器
            driver.quit();
        }
}
```
## 2.获取与删除 Cookie
### （1）获取指定 Cookie
```java
public static void main(String[] args) {
       WebDriver driver = new ChromeDriver();
        try {
            driver.get("http://www.example.com");
            //设置一个Cookie
            driver.manage().addCookie(new Cookie("login", "fgflkshf&"));
            // 获取key是 'login'的Cookie
            Cookie cookie1 = driver.manage().getCookieNamed("login");
            System.out.println(cookie1);
        } finally {
            driver.quit();
        }
}
```
### （2）获取所有 Cookie
```java
		Set<Cookie> cookies = driver.manage().getCookies();
```

### （3）删除指定 Cookie
```java
 		driver.manage().deleteCookieNamed("login");
```
### （4）删除所有 Cookie
```java
		driver.manage().deleteAllCookies();
```


# 八、键盘与鼠标操作

键盘与鼠标操作可参考官网说明：

1.[键盘操作说明](https://www.selenium.dev/documentation/webdriver/actions_api/keyboard/)

2.[鼠标操作说明](https://www.selenium.dev/documentation/webdriver/actions_api/mouse/)

3.[滚轮操作说明](https://www.selenium.dev/documentation/webdriver/actions_api/wheel/)

----

参考：
[Java-Selenium自动化教程(学了不亏)](https://blog.csdn.net/weixin_45203607/article/details/125895112)
[JAVA使用selenium的常见爬虫操作](https://blog.csdn.net/shaomeng7426/article/details/125820368)
[Selenium with Python](https://selenium-python.readthedocs.io/)
[Selenium WebDriver](https://www.javatpoint.com/selenium-webdriver)
[selenium 使用使用已经打开的浏览器](https://www.cnblogs.com/testway/p/16676195.html)