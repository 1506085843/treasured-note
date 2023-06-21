[TOC]

# 前言
**为什么要有单元测试？**

举个例子，如果你写了一个比较复杂的方法，这个函数接收两个参数然后计算会输出一个值，经过一系列测试没问题。

可是后来因为功能变化，别人又修改了这个方法，你又需要再把以前测试的数据再测一遍，另外如果测试不全面的话，那这个方法就会有问题。而且如果以果这个方法又改动了，那岂不是又要再全部测试一次？
单元测试就可以把你准备好的所有测试用例，在每次测试或要上线时运行下看符不符合预期的结果，不符合就直接报错，可以做到快速测试并发现问题。

# 一、JUnit 4与JUnit 5区别

JUnit 5 旨在适应[Java 8](https://howtodoinjava.com/java-8-tutorial/) 编码风格，并比 [JUnit 4](https://howtodoinjava.com/junit-4/) 更健壮和灵活。

JUnit4 需要Java5 或以上版本，JUnit4把所有的代码都打包到一个jar包。

使用 JUnit5 需要 Java 8 或更高版本，JUnit5由三个子项目构成：JUnit平台(JUnit Platform)，JUnit Jupiter和JUnit Vintage。

- JUnit Platform:它定义了测试引擎（TestEngine）API，用于开发运行在JUnit平台上面的新的测试框架。
- JUnit Jupiter：它拥有所有的新的JUnit注解和测试引擎的实现（Implementation），这个测试引擎的实现能够测试使用新注解开发的测试代码。
- JUnit Vintage：用于支持在JUnit5平台上运行JUnit3和JUnit4编写的测试用例

此外主要的区别是断言Assert和其他注解以及使用上的不同，例如：使用JUnit 5 测试类上可以不用写 @RunWith 。

具体其他不同点可以参考 [JUnit4和JUnit5的主要区别](https://blog.csdn.net/lzufeng/article/details/127521842)

# 二、新建一个maven项目

先按照这篇文章使用idea的maven模板创建一个maven项目：[idea快速创建maven项目](https://blog.csdn.net/qq_33697094/article/details/129525957)

# 三、pom文件配置


## 1.引入junit5依赖

```xml
  <dependencies>
    <!-- junit5用于单元测试-->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.9.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

上面使用的junit-jupiter依赖是Junit 提供的一个 集合器(aggregator)：它 集成了其他需要的 Junit 依赖，包括：
- junit-jupiter-api

- junit-jupiter-params

- junit-jupiter-engine

这样你就只需写集合器那一个依赖就可以了。

## 2.引入 maven-surefire-plugin插件

引入 [maven-surefire-plugin](https://so.csdn.net/so/search?q=maven-surefire-plugin&spm=1001.2101.3001.7020) 插件以支持 Maven 执行应用程序的单元测试并生成txt或xml格式的报告。
（默认情况下，报告生成在`{项目目录}/target/surefire-reports/TEST-*.xml`）。

```xml
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
```

## 3.最终的pom.xml

因为用的idea的maven模板创建的maven项目，其最终生成的pom.xml中 pluginManagement里的插件版本可能有点旧，你可以自己指定插件版本，也可以把用不到的插件删除。

最终你的pom.xml看起来像下面这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>mavenTest</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>mavenTest</name>

  <!-- 指定项目编译使用的jdk版本-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <!-- junit5用于单元测试-->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.9.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <pluginManagement>
      <plugins>
        <!-- clean 清除构建在target文件夹下生成的各个Jar包和class等文件-->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.2.0</version>
        </plugin>

        <!-- 用于编译项目的源代码 -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.11.0</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
            <encoding>UTF-8</encoding>
          </configuration>
        </plugin>

        <!-- 支持单元测试 -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>

        <!-- 用于将项目打包成一个jar -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.3.0</version>
        </plugin>

        <!-- 将项目生成的jar包添加到本地存储库,方便其他项目引用。（其他项目通过groupId、artifactId、version来引用本项目的jar） -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-install-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

## 4.扩展

生命周期、生命周期阶段、插件和插件目标是 Maven 的核心。

- Maven 命令**mvn**只接受生命周期阶段或插件目标作为参数。
- Maven 带有三个生命周期：default、clean 和 site。
- **每个生命周期由生命周期阶段组成**，总共有 28 个阶段——**默认 生命周期包含21 个阶段**（*验证、...、编译、...、打包、...、安装、部署*），**清理的生命周期含有3个阶段**（*预编译）清洁、清洁、清洁后*），**站点生命周期含有 4 个**（*站点前、站点、站点后、站点部署*）。
- 当使用命令调用生命周期阶段时`mvn`，所有前面的阶段都会依次执行。
- 生命周期阶段本身不具备完成任何任务的能力，它们依赖于插件来执行任务。
- 根据项目和打包类型，**Maven 将各种插件目标绑定到生命周期阶段**，目标执行委托给它们的任务。



> IDEA 主界面右侧 Maven页签的 Lifecycle 和 Plugins 里有同样的命令，比如 install，既在 Plugins 中存在，也在 Lifecycle中存在，它们有什么区别吗？
>
> 简单来说Lifecycle 下的是执行整个生命周期，而Plugins 下的是只执行它具体的阶段。
>
> 举个例子：
>
> 你点击Lifecycle 下的  install :
>
> 它会帮你执行如下过程  validate --> compile --> test --> package --> integration-test --> verify --> install ，也就是说它会帮你执行验证、编译、单元测试、打包、整合测试、最后再把生成的jar包发布到本地仓库，换句话说它会帮你执行 install 之前的所有操作，最后执行 install 
>
> 你点击 Plugins 下的 install :
>
> 它只会帮你把生成的jar包发布到本地仓库，也就是说它只会帮你执行 install ，其前面的操作它不会帮你执行。
>
> 感兴趣的话可以看下这篇文章：[IDEA 中 maven 的 Lifecycle 和Plugins 的区别](https://www.jb51.net/article/276738.htm)

