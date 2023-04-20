[TOC]
# 前言
最近在处理一些音频数据，ffmpeg是一款非常好用处理音视频的工具包。那什么是ffmpeg呢？FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序，可以结合Java开发一些处理视频音频的功能。

# 1.ffmpeg下载
首先打开 [ffmpeg官网下载](https://www.ffmpeg.org/download.html#build-windows)
或者用 [百度云](https://pan.baidu.com/s/1dCK-TrOcUfC6pdKi2Y1e6g) 下载（https://pan.baidu.com/s/1dCK-TrOcUfC6pdKi2Y1e6g  提取码：2pdo）

然后点击 windows 对应的图标，再点击下面的”Windows EXE File”随便选一个点进去选择一个版本下载。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011617461123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)

# 2.下载后解压，配置环境变量
下载解压后就能在 bin 文件夹下能看到三个可执行程序：ffmpeg、ffplay、ffprobe，配置好环境变量后即可使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116175104399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)
验证是否成功：
cmd窗口输入ffmpeg -version 。如下图则安装成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011617515611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)

# 3.介绍FFmpeg组成

构成FFmpeg主要有三个部分

**3.1第一部分是四个作用不同的工具软件，分别是：**
- ffmpeg.exe：音视频转码、转换器
- ffplay.exe：简单的音视频播放器
- ffprobe.exe：简单的多媒体码流分析器


**3.2第二部分是可以供开发者使用的SDK，为各个不同平台编译完成的库。**
如果说上面的四个工具软件都是完整成品形式的玩具，那么这些库就相当于乐高积木一样，我们可以根据自己的需求使用这些库开发自己的应用程序。这些库有：

libavcodec：包含音视频编码器和解码器
libavutil：包含多媒体应用常用的简化编程的工具，如随机数生成器、数据结构、数学函数等功能
libavformat：包含多种多媒体容器格式的封装、解封装工具
libavfilter：包含多媒体处理常用的滤镜功能
libavdevice：用于音视频数据采集和渲染等功能的设备相关
libswscale：用于图像缩放和色彩空间和像素格式转换功能
libswresample：用于音频重采样和格式转换等功能

**3.3第三部分是整个工程的源代码，无论是编译出来的可执行程序还是SDK，都是由这些源代码编译出来的。**
FFmpeg的源代码由C语言实现，主要在Linux平台上进行开发。FFmpeg不是一个孤立的工程，它还存在多个依赖的第三方工程来增强它自身的功能。在当前这一系列的博文/视频中，我们暂时不会涉及太多源代码相关的内容，主要以FFmpeg的工具和SDK的调用为主。到下一系列我们将专门研究如何编译源代码并根据源代码来进行二次开发。


# 4.简单使用
比如，使用ffmpeg获取视频的一些信息：
>ffprobe -show_format D:\507-#网愈云故事收藏馆.mp4

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116200220529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)

播放音频文件的命令：

>ffplay D:\507-#网愈云故事收藏馆.mp4

这时候就会弹出来一个窗口，一边播放MP3文件，一边将播放音频的图画到该窗口上。针对该窗口的操作如下：

点击该窗口的任意一个位置，ffplay会按照点击的位置计算出时间的进度，然后seek到计算出来的时间点继续播放。
按下键盘的左键默认快退10s，右键默认快进10s，上键默认快进1min，下键默认快退1min。
按ESC就退出播放进程，按W会绘制音频的波形图。


# 5.使用Java调用ffmpeg，进行音视频的转换、音视频提取、音视频截取
参考我另一篇文章，代码可直接使用:
- [ffmpeg进行音视频格式转换、合并、播放、截图](ffmpeg进行音视频格式转换、合并、播放、截图.md)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012616295943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)


----

其他的一些ffmpeg命令行使用可以参考：
[FFmpeg 的介绍与使用](https://blog.csdn.net/u011330638/article/details/82392268)
[ffmpeg命令详解](https://blog.csdn.net/zzcchunter/article/details/68060989)

参考文章：
[FFmpeg安装（windows环境）](https://www.cnblogs.com/xiezhidong/p/6924775.html)
[总结FFMPEG视音频编解码零基础学习方法](https://blog.csdn.net/leixiaohua1020/article/details/15811977/)
[FFmpeg命令行工具学习(二)：播放媒体文件的工具ffplay](https://www.cnblogs.com/renhui/p/8458802.html)