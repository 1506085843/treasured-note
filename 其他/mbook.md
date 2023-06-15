[toc]

# 介绍

**mdBook**是一个使用 Markdown 创建书籍的命令行工具。它非常适合创建产品或 API 文档、教程、课程材料或任何需要清晰、易于导航和可定制的演示文稿。

- 轻量级[Markdown](https://rust-lang.github.io/mdBook/format/markdown.html)语法可帮助您更专注于您的内容
- 综合[搜索](https://rust-lang.github.io/mdBook/guide/reading.html#search)支持
- 许多不同语言的代码块的颜色[语法突出显示](https://rust-lang.github.io/mdBook/format/theme/syntax-highlighting.html)
- [主题](https://rust-lang.github.io/mdBook/format/theme/index.html)文件允许自定义输出格式
- [预处理器](https://rust-lang.github.io/mdBook/format/configuration/preprocessors.html)可以为自定义语法和修改内容提供扩展
- [后端](https://rust-lang.github.io/mdBook/format/configuration/renderers.html)可以将输出呈现为多种格式
- 为了速度、安全和简单而用[Rust编写](https://www.rust-lang.org/)
- [Rust 代码示例](https://rust-lang.github.io/mdBook/cli/test.html)的自动化测试

mdBook 的官方 github 地址：[mdBook](https://github.com/rust-lang/mdBook)

# 下载与创建项目

## 1.下载

根据你的操作系统，去  [mdBook发行](https://github.com/rust-lang/mdBook/releases) 页面下载最新的 mdBook 包。

或者

[百度云windos版下载](https://pan.baidu.com/s/1nd1343_nmaZBu_ogcWaN1Q )（链接： https://pan.baidu.com/s/1nd1343_nmaZBu_ogcWaN1Q 提取码：isjq ）

我这里下载的是 mdbook-v0.4.30-x86_64-pc-windows-msvc.zip 。

## 2.初始化

将压缩包解压后，在 mdbook.exe 所在文件夹里打开 cmd 命令行， 执行如下命令进行初始化：

```bash
mdbook.exe init ./mybook
```

初始化的时候给你的树或项目起个名字，上面我起的是 mybook ，然后它会问你要不要创建 .gitignore 和 标题目录，可根据需要输入，最后会生成如下项目目录：

```bash
mybook/
├── book
├── book.toml
└── src
    ├── chapter_1.md
    └── SUMMARY.md
```

## 3.结构说明

- book.toml 是你的配置文件，可以配置每篇文章最上面统一标题、路径、作者等信息。

- 你新增的 Markdown文章都要放到 src 目录下。

- SUMMARY.md 是你所有文章的目录，你新增的 Markdown文章都需要在这里面配置下。

- book 文件夹不用管，最后 mdBook 启动后会自动生成一些 html 文件在里面。

# 编写文章与启动

## 1.编写文章

现在你就可以在 src 目录下编写自己的  Markdown文章，然后把文章名在 SUMMARY.md 目录文件里添加下就可以了。

## 2.构建

使用如下构建命令，便可生成在你项目的 book 目录下生成最终的一些 html 文件：

```bash
mdbook.exe build D:\Program\mdbook\mybook\
```

你可以打开 book 目录下的 index.html 查看效果，你还可以将 book 目录放到你服务器上然后映射到 nginx 目录下，并配置虚拟主机就拥有了自己的电子书或者博客文档啦。

## 3.启动 mdbook 服务

你也可以启动 mdbook 服务：

```bash
mdbook.exe serve D:\Program\mdbook\mybook\
```

启动时它会先执行构建命令，然后开启 3000 端口，启动成功后，你就可以浏览器访问 `http://localhost:3000`，便可本地看到效果了。

效果如下：

![](https://img-blog.csdnimg.cn/028204d376ee49eda169258a5e6dfbea.jpeg)


# 其他配置

关于 mdBook 的其他配置你可以参考官方操作文档： [mdBook 文档](https://rust-lang.github.io/mdBook/index.html)