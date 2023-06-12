[TOC]


# 前言
git有很多操作并且命令很多，本文总结了常用的git操作情况以便使用git遇到问题时快速查询解决

# 一、删除文件夹
例1：删除.idea文件夹：


```bash
$ git --help                            # 帮助命令
$ git pull origin master               # 将远程仓库里面的项目拉下
$ dir                                # 查看有哪些文件
$ git rm -r --cached .idea          # 删除.idea文件夹
$ git commit -m '删除.idea'        # 提交,添加操作说明
$ git push -u origin master      # 将本次更改更新到github项目master分支
```


例2：删除项目servic 文件夹下build文件夹及其中所有文件：

```bash
$ git rm -r --cached  servic/build      # 删除servic下的build 文件夹
$ git commit -m '删除build '            # 提交,添加操作说明
$ git push -u origin master            # 将本次更改更新到github项目master分支
```
-------------------------------------------------------------------------------------------------------
# 二、删除文件

例：删除项目跟目录下的文件mvnw.cmd：

```bash
$ git rm -f --cached mvnw.cmd          # 删除mvnw.cmd文件
$ git commit -m '删除mvnw '            # 提交,添加操作说明
$ git push -u origin master           # 将本次更改更新到github项目master分支
```

-----------------------------------------------------------------------------------------------------------

# 三、修改文件夹名
例：将文件夹名称:game  修改为: gamesdk

```bash
$ git mv game gamesdk                              #修改文件名
$ git commit -m 'rename dir game to gamesdk'      #提交,添加操作说明
$ git push origin dev                            # 推送到dev 分支
```
-----------------------------------------------------------------------------------------------------------
# 四、忽略文件
忽略某一文件，比如当我们要add时发现编译器产生了一些文件可以通过以下命令忽略
例：
```bash
$ git update-index --assume-unchanged logs/*.log         # 忽略logs文件夹下所有log后缀结尾的文件
```

-----------------------------------------------------------------------------------------------------------
# 五、版本回退
1.git版本回退到上一版本
例：

```bash
$ git  log                    # 查看最近修改提交记录
$ git  reset --hard  HEAD^   #回退到上个版本
```
-----------------------------------------------------------------------------------------------------------
2..git版本回退到某一版本：
例：

```bash
$ git  log               # 查看最近修改提交记录
$ git reset  --hard A   #回退到上个版本，A为想要回滚的版本号
$ git reset  A         #将A移到最新的版本
```
-----------------------------------------------------------------------------------------------------------
# 六、查看某一次提交了哪些文件
例：
```bash
$ git  log               # 查看最近修改提交记录
$ git show A --stat       #回退到上个版本，A为想要查看的版本号
```

-----------------------------------------------------------------------------------------------------------
# 七、查看某一文件的提交历史记录
```bash
$ git  log  filename   
```
-----------------------------------------------------------------------------------------------------------
# 八、查看某人的历史提交记录
如查看xiaoming的提交记录
例：
```bash
$ git log --author="xiaoming" 
```

-----------------------------------------------------------------------------------------------------------
# 九、本地代码文件还原
如果你修改了A.java文件，还没有commit和push，此时如果你想还原A.java文件的修改可如下操作：
```bash
$ git  checkout   A.java            # 还原A.java文件的修改
```
-----------------------------------------------------------------------------------------------
# 十、暂存区里的文件移出暂存区
git将add到暂存区里的文件移出暂存区，即有时候你add的时候加错了文件到缓存区可如下将文件移出缓存区：
```bash
$ git rm --cache B.java           # 将B.java文件移出缓存区，代码修改不会被还原
```

如果某一目录下的多个文件都被错误的加入了缓存区可如下将目录下所有文件移出缓存区：
```bash
$ git rm --cache -r mbos-portal/webContent/.history/         # 将文件夹下的文件批量移出缓存区，mbos-portal/webContent/.history/ 是文件夹名
```

-----------------------------------------------------------------------------------------------
# 十一、本地分支关联远程分支
如果你是在本地新建了一个test分支，远程并没有test分支，你现在想把本地的test分支推送到远程
```bash
$ git push --set-upstream origin test  #也可以使用这种简写形式： git push -u origin test
```

-----------------------------------------------------------------------------------------------------------
# 十二、创建分支
创建dev分支：

```bash
$  git branch  dev
```
-------------------------------------------------------------------------------------------------------
# 十三、分支切换
1.创建panda分支并切换到panda分支：

```bash
$ git checkout -b panda                   #创建并切换
$ git push --set-upstream origin panda  #创建的panda  分支提交到远程
```

2.切换到master分支：

```bash
$ git checkout master
```
--------------------------------------------------------------------------------------------------------
# 十四、查看所有分支

```bash
$ git branch -a      
```

-------------------------------------------------------------------------------------------------------
# 十五、查看当前所在分支
```bash
$ git branch   
```
-------------------------------------------------------------------------------------------------------
# 十六、删除本地的分支
删除本地的dev分支：

```bash
$ git branch -d dev
```
--------------------------------------------------------------------------------------------------------
# 十七、删除远程的分支
删除远程的dev分支：

```bash
$ git push origin --delete dev
```
--------------------------------------------------------------------------------------------------------
# 十八、分支合并
1.dev分支合并到master分支：
 如果我们在dev上开发了代码，要把dev的代码合并到master上：

```bash
$ git checkout master                 #先切换到master分支
$ git merge dev                       #将dev分支代码合并到当前分支
$ git commit -m "合并分支"             # 提交,添加操作说明
$ git push -u origin master           # 将本次更改更新到master分支
```

2.如果是idea上可如下图形化操作来分支合并：
（1）在dev分支上拉取最新代码
（2）切换到master分支
	![在这里插入图片描述](https://img-blog.csdnimg.cn/b845f34da63f4741b92f7e76b2fcf7dd.png#pic_center)
（3）点击dev选择将dev合并到master分支
![在这里插入图片描述](https://img-blog.csdnimg.cn/95f60ac5542d48b886dcf0d6ceb88bc2.png#pic_center)
（4）如果有冲突解决冲突的文件

（5）Terminal 命令行输入 git push 提交推送到远程分支
![在这里插入图片描述](https://img-blog.csdnimg.cn/0561f663f35040d8a04b784466d4ae45.png)

-----
# 十九、文件对比


（1）.如果修改了多个文件，并且多个文件都没有使用git add加入到缓存区，那么可以使用git diff命令，会列出这些文件所有修改的地方

```bash
git diff
```

（2）.如果某文件没有使用git add加入到缓存区，那么可以使用git diff <filename> 命令,列出指定文件所有修改的地方

```bash
git diff demo/Test.java
```

（3）.比较某次提交和工作区的demo/Test.java文件的不同。XXXX是版本号

```bash
git diff XXXX demo/Test.java
```


（4）.如果多个文件已经使用了git add加入到了缓存区，使用下面的命令会列出这些文件所有修改的地方

```bash
git diff --cached
```

（5）.如果某个文件已经使用了git add加入到了缓存区，使用下面的命令会列出该文件所有修改的地方

```bash
git diff --cached demo/Test.java
```


（6）.查看当前工作区内容与XXXX版本号的所有文件内容的差异

```bash
git diff XXXX   #XXXX是版本号
```


（7）.比较两个版本号所有文件差异

```bash
git diff XXXX1 XXXX2   #XXXX1和XXXX2是版本号
```

（8）.除了上面的使用，还可以很方便的对比两个文件的差异
（对比电脑 D 盘的 revised.txt 和 original.txt 的差异）
```bash
 git diff --no-index D:/revised.txt D:/original.txt
```

-----
# 二十、cherry-pick操作
dev分支上依次有A、B、C、D 4次提交，每次提交了许多改动的文件，现在需要将B、D两次提交的改动也同步到master分支：

```bash
$ git checkout master                #先切换到master分支
$ git cherry-pick  XXXX             #XXXX是B的版本号
$ git cherry-pick  XXXX             #XXXX是D的版本号
```

#xxx是版本号，如：git cherry-pick 7c8afdf6       
如果没有冲突会自动的提交到master分支，有冲突则手动解决冲突再手动提交
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916173623725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70#pic_center)

---

# 二十一、切换到指定tag分支
一个项目不仅会有多个分支，还可能会有多个tag
当clone一个项目后，如果想切换到指定tag可以如下操作：

（1）.查看所有的tag名称

```bash
git tag -l
```

（2）.新建一个分支该分支使用指定tag。
假如新建一个dev2分支，该分支采用1.1.0代码

```bash
git checkout tags/dev2 -b 1.1.0
```

（3）.切换到指定tag。假如要切换到1.1.0的tag

```bash
git checkout tags/1.1.0
```


# 二十二、代码暂存

如果本地代码没开发完成，这个时候去拉取最新的代码可能会有冲突；或者本地代码没开发完，此时要切换分支可以使用暂存功能。
```bash
git stash
```
这样本地的所有修改就都被暂时存储起来了包括git add加到缓存区里的代码 。其中 **stash@{0}** 就是刚才保存的标记。后续可以通过此标记访问。


当你拉取更新了代码后，可以通过如下操作还原暂存的内容：
```bash
git stash pop stash@{0}
```

如果暂存区不用了可以删除暂存区：
```bash
git stash drop stash@{0}
```

# 二十三、当前分支的所有文件中查找字符串
在当前分支的所有文件中查找字符串 cullateSum 出现在哪些文件的哪几行
```bash
git grep -n "cullateSum" 
```

# 二十四、所有分支的所有文件中查找字符串
（1）在所有远程分支的所有文件中查找字符串 cullateSum 出现在哪些文件的哪几行
```bash
git grep -n "cullateSum" $(git for-each-ref --format='%(refname)' refs/remotes)
```
（2）在所有本地分支的所有文件中查找字符串 cullateSum 出现在哪些文件的哪几行
```bash
git grep -n "cullateSum" $(git for-each-ref --format='%(refname)' refs/heads)
```

# 二十五、已提交的删除文件进行恢复
假设很久以前别人删除了几个文件并提交到了远程分支，如果现在你想还原被删除的文件或者想复制下那些被删除的文件，可以按照如下操作：

idea 里找到很久以前别人的那次提交记录，选择你要还原的文件，然后右键选择Revert Selected Changes ,这时候这些文件就被还原到了你本地，这个时候你就可以进行文件的复制或者提交操作。

![请添加图片描述](https://img-blog.csdnimg.cn/6c518b68eb014fa18f94b5e92103f039.png)

![请添加图片描述](https://img-blog.csdnimg.cn/178c1355a3864c98b05765c632a3b28c.png)
# 二十六、查看当前分支是基于哪个分支创建的

```bash
git reflog --date=local | grep 你的分支名
```

# 二十七、本地项目上传 github
如果你有一个项目，想把它上传到github，可参照我另一篇文章：[本地项目上传github](https://blog.csdn.net/qq_33697094/article/details/111995819)

# 二十八、查询某一文件的删除时间和删除人
如果你不知道某一文件是谁在什么时候删除的，想查询该文件的删除时间、删除人以及提交id，可使用如下命令：
```bash
git log --full-history -- 你的文件名
```
例如：
查找 datamodel/fi/1.5/main/ar/metadata/sm_salorder_ar_finarbill_1669423033909277696.cr 的删除时间
```bash
git log --full-history -- datamodel/fi/1.5/main/ar/metadata/sm_salorder_ar_finarbill_1669423033909277696.cr
```
结果：
第一条便是删除的时间、删除人等信息

![请添加图片描述](https://img-blog.csdnimg.cn/ea5f7534efc6415188bc81714ae304ea.png)

---

参考：
[git忽略已经被提交的文件](https://segmentfault.com/q/1010000000430426)
[Hudson error when checking out from Git](https://stackoverflow.com/questions/791959/download-a-specific-tag-with-git)
[Is it possible to perform a 'grep search' in all the branches of a Git project](https://stackoverflow.com/questions/15292391/is-it-possible-to-perform-a-grep-search-in-all-the-branches-of-a-git-project/21284342#21284342)
[Using Git, how could I search for a string across all branches](https://stackoverflow.com/questions/7151311/using-git-how-could-i-search-for-a-string-across-all-branches)