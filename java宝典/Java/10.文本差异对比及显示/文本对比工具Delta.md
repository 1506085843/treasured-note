@[TOC](目录)

# 介绍

## git diff 介绍

平时我们会在命令行使用 `git diff` 相关命令来对比文件的差异。（git diff命令可参考：[git-diff命令说明](https://git-scm.com/docs/git-diff)）

比如：当前文件和该文件以前某一次提交进行对比、某文件的某两次提交记录进行对比、对比电脑上任意两个文件的差异。

但是 git diff  对比出来没有行号，并且不能并排显示差异，对比界面不太好看，，这个时候你就可以使用 delta 了 。

## delta介绍

delta 是一款用于命令行的对比开源工具，它是基于 git 的，也就是它是通过  git 来对比文本得到不同点，然后做了美化处理，delta 提供了许多主题和配置，可以让你在命令行很方便的对比代码或文本，以此来提高你的工作效率。

所以如果你也经常使用命令行以及使用命令行来对比，那么 delta 就很适合你。

[delta的github官网](https://github.com/dandavison/delta)


# 一、安装

## 1.下载 Git

delta 是基于 Git 的，如果你没安装 Git 需要先去 [Git官网下载](https://git-scm.com/downloads) 安装下 Git。

## 2.下载 delta

根据你的操作系统，到 delta 的发行下载页面下载最新的版本，传送门：[delta下载](https://github.com/dandavison/delta/releases)，（因为我的电脑操作系统是 windos 的，所以我下载的是 delta-0.16.5-x86_64-pc-windows-msvc.zip）

## 3.解压

下载解压后，把 delta.exe 放到如下目录：

```bash
C:\Users\你的用户名\AppData\Local\Microsoft\WindowsApps
```

## 4.修改配置文件

打开 `C:\Users\用户名\.gitconfig` 文件，在其中增加如下配置：

```bash
[core]
	pager = delta

[interactive]
    diffFilter = delta --color-only

[delta]
    navigate = true		#运行使用 n 和 N 在 diff 部分之间移动
    line-numbers = true  #行号
    side-by-side = true	 #并排对比视图
    syntax-theme = Coldark-Cold #主题

[merge]
    conflictstyle = diff3

[diff]
    colorMoved = default
```

（如果你该目录下没有 .gitconfig 文件需要自己新建一个）

## 5. 修改主题

上面我们使用了Coldark-Cold 主题，它是为暗色背景提供的一个主题。
delta 为亮色和暗色背景的命令行提供了多种主题，你可以使用 `delta --list-syntax-themes` 命令来查看所有的主题。

   ```bash
亮色背景有以下:
    GitHub
    Monokai Extended Light
    OneHalfLight
    Solarized (light)
    gruvbox-light

暗色背景有以下:
    1337
    Coldark-Cold
    Coldark-Dark
    DarkNeon
    Dracula
    Monokai Extended
    Monokai Extended Bright
    Monokai Extended Origin
    Nord
    OneHalfDark
    Solarized (dark)
    Sublime Snazzy
    TwoDark
    Visual Studio Dark+
    ansi
    base16
    base16-256
    gruvbox-dark
    zenburn
   ```


## 6.其他配置和说明

关于delta 的其他配置和自定义主题颜色等操作，可参考 ：[delta 官方配置说明](https://dandavison.github.io/delta/configuration.html)
# 二、对比命令

## 1.在项目中 git diff 常用命令

（1）.如果修改了多个文件，并且多个文件都没有使用 git add 加入到缓存区，那么可以使用 git diff 命令，会列出这些文件所有修改的地方

```bash
git diff
```

（2）.如果 Test.java 文件没有使用 git add 加入到缓存区，那么可以如下列出该文件所有修改的地方

```bash
git diff Test.java
```

（3）.比较某次提交和工作区的 Test.java文件的不同，XXXX 是 commitId

```bash
git diff XXXX Test.java
```


（4）.如果多个文件已经使用了git add加入到了缓存区，使用下面的命令会列出这些文件所有修改的地方

```bash
git diff --cached
```

（5）.如果某个文件已经使用了git add加入到了缓存区，使用下面的命令会列出该文件所有修改的地方

```bash
git diff --cached demo/Test.java
```


（6）.查看当前工作区内容与 某次提交 的所有文件内容的差异

```bash
git diff XXXX   #XXXX是 commit Id
```


（7）.比较两个版本号所有文件差异

```bash
git diff XXXX1 XXXX2   #XXXX1和XXXX2是 commit Id
```

## 2.对比电脑上两个文件

- 对比 revised.txt 和  original.txt 并显示他们的差异（只显示差异不同点）：

```bash
git diff --no-prefix revised.txt original.txt
或者：
detal revised.txt original.txt
```

- 对比 revised.txt 和  original.txt 显示差异和文本所有内容：

```bash
git diff --no-prefix -U99999 revised.txt original.txt
```

## 3.对比电脑上的两个文件夹

你可以使用如下命令对比两个文件夹下所有文件的差异 （dir1 和 dir2 是你文件夹的名称）

```bash
detal dir1 dir2
```

# 三、在Git 命令行中使用效果

**1.Git 命令行中Coldark-Cold 主题的效果**

![请添加图片描述](https://img-blog.csdnimg.cn/0854dcd0be494ffbad92c0bffbf8e230.jpeg)

**2.Git 命令行中 GitHub 亮色主题的效果**

在Git 命令行中如果你想使用亮色的 GitHub 主题，你需要先把你的命令行背景颜色改为白色，
鼠标右键 ==> Options ==> looks  ==> Background

![请添加图片描述](https://img-blog.csdnimg.cn/c0fb7749a2144fb28a8d3c4bd775f050.jpeg)

![请添加图片描述](https://img-blog.csdnimg.cn/18b86042fd4649b89b7b35556efab292.jpeg)

# 四、在idea 的Terminal命令行中使用效果

**1.使用 Dracula 主题在idea 的Terminal命令行中效果：**

![请添加图片描述](https://img-blog.csdnimg.cn/041efaec5e8443bebc7af1071cdfdf15.jpeg)


**2.使用 GitHub 主题在idea 的Terminal命令行中效果：**

（GitHub 主题是亮色主题，需要把 idea 的背景设置成白色，idea 里左上角 File --> Setting -->【Appearance & Behavior】下的 Appearance  --> Theme下拉框选择IntelliJ Light）

![请添加图片描述](https://img-blog.csdnimg.cn/6f2e09d420694481962e0ab956c408c4.jpeg)
