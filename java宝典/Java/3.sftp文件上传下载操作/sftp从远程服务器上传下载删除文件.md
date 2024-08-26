[TOC]


## 前言
有时候我们需要让从代码里远程连接服务器进行文件上传、下载、判断文件路径是否存在、创建文件夹等操作。

这时候就用到了sftp。常见的三个库是：JSch、SSHJ 和 Apache Commons VFS它们都能实现远程连接服务器。

本文主要讲解利用JSch远程连接服务器。


## 一、所需依赖
maven如下：
```xml
        <dependency>
            <groupId>com.jcraft</groupId>
            <artifactId>jsch</artifactId>
            <version>0.1.55</version>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>
```

## 二、工具类

SftpUtil 工具类如下：
```java
package com.hai.tang.util;

import com.jcraft.jsch.*;
import org.apache.commons.io.IOUtils;

import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.util.Properties;

/**
 * 用于连接远程服务器
 */
public class SftpUtils {
    private static JSch jsch;
    private static Session session = null;
    private static ChannelSftp channelSftp = null;

    //服务器用户名
    private String user;

    //服务器密码
    private String password;

    //服务器ip
    private String host;

    //服务器端口
    private int port;

    public SftpUtils() {
    }

    public SftpUtils(String host, int port, String user, String password) {
        this.user = user;
        this.password = password;
        this.host = host;
        this.port = port;
    }

    /**
     * 开启连接
     */
    public ChannelSftp connect() {
        jsch = new JSch();
        try {
            // 根据用户名、主机ip、端口号获取一个Session对象
            session = jsch.getSession(user, host, port);
            // 设置密码
            session.setPassword(password);
            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            // 为Session对象设置properties
            session.setConfig(config);
            // 设置连接超时为5秒
            session.setTimeout(100 * 50);
            // 通过Session建立连接
            session.connect();
            // 打开SFTP通道
            channelSftp = (ChannelSftp) session.openChannel("sftp");
            // 建立SFTP通道的连接
            channelSftp.connect();
        } catch (JSchException e) {
            e.printStackTrace();
        }
        return channelSftp;
    }

    /**
     * 关闭连接
     */
    public void close() {
        if (channelSftp != null) {
            try {
                channelSftp.disconnect();
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
        if (session != null) {
            try {
                session.disconnect();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 判断文件夹路径是否存在
     *
     * @param directory 文件夹路径，如：/root/test/saveFile/
     */
    public boolean isDirExist(String directory) {
        directory = null != directory && directory.endsWith("/") ? directory : directory + "/";
        boolean dirExist = false;
        try {
            SftpATTRS sftpAttrs = channelSftp.lstat(directory);
            dirExist = sftpAttrs.isDir();
        } catch (Exception e) {
            if ("no such file".equalsIgnoreCase(e.getMessage())) {
                return false;
            }
        }
        return dirExist;
    }

    /**
     * 创建一个文件夹(若整个路径都不存在会依次创建，若该路径已经存在则不会创建)
     *
     * @param createpath 要创建的文件夹路径，如：/root/test/saveFile/
     */
    public void createDir(String createpath) {
        createpath = null != createpath && createpath.endsWith("/") ? createpath : createpath + "/";
        if (!isDirExist(createpath)) {
            StringBuilder builder = new StringBuilder("/");
            String[] pathArry = createpath.split("/");
            for (String dir : pathArry) {
                if (!"".equals(dir)) {
                    builder.append(dir);
                    builder.append("/");
                    try {
                        String path = builder.toString();
                        if (!isDirExist(path)) {
                            // 建立目录
                            channelSftp.mkdir(path);
                        }
                    } catch (SftpException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    /**
     * 删除文件
     *
     * @param deleteFile 要删除的文件路径，如:/root/test/saveFile/mylog.log
     */
    public void deleteFile(String deleteFile) {
        try {
            channelSftp.rm(deleteFile);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 文件上传
     *
     * @param fileStram 文件输入流
     * @param upToPath  要上传到的文件夹路径
     * @param fileName  上传后的文件名
     */
    public void uploadFile(InputStream fileStram, String upToPath, String fileName) {
        upToPath = null != upToPath && upToPath.endsWith("/") ? upToPath : upToPath + "/";
        try {
            channelSftp.put(fileStram, upToPath + fileName);
        } catch (SftpException e) {
            e.printStackTrace();
        }
    }

    /**
     * 文件下载
     *
     * @param downlownPath 要下载的文件的所在文件夹路径
     * @param fileName     文件名
     * @return download  返回下载的文件流
     */
    public InputStream downloadFile(String downlownPath, String fileName) {
        downlownPath = null != downlownPath && downlownPath.endsWith("/") ? downlownPath : downlownPath + "/";
        InputStream download = null;
        try {
            download = channelSftp.get(downlownPath + fileName);
        } catch (SftpException e) {
            e.printStackTrace();
        }
        return download;
    }


    /**
     * 执行linux命令
     *
     * @param order 要执行的命令，（如，打印指定目录下的文件信息: ls -a /usr/local/kkFileView/kkFileView-4.0.0/bin/）
     * @return result  执行后返回的结果
     */
    public String excutOrder(String order) {
        String result = "";
        try {
            ChannelExec channelExec = (ChannelExec) session.openChannel("exec");
            channelExec.setCommand(order);
            channelExec.setErrStream(System.err);
            channelExec.connect();
            InputStream in = channelExec.getInputStream();
            result = IOUtils.toString(in, Charset.defaultCharset());
        } catch (JSchException | IOException e) {
            e.printStackTrace();
        }
        return result;
    }

}

```

如果你使用 springboot，可修改上面代码，使用户名、密码、服务器ip、端口从 yaml 或 properties 配置文件直接读取：

```java
    @Value("${c.test.uploadFile.username:root}")
    private String ftpUserName;//用户名

    @Value("${c.test.uploadFile.password:123546:}")
    private String ftpPassword;//密码

    @Value("${c.test.uploadFile.host:127.0.0.7}")
    private String ftpHost;//服务器ip

    @Value("${c.test.uploadFile.port:22}")
    private String ftpPort;//端口
    
```

## 三、测试
### 1.判断指定目录是否存在
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //判断目录是否存在
        boolean dirBoolean = sftpUtil.isDirExist("/usr/local/aaa/");
        //关闭连接
        sftpUtil.close();
}
```


### 2.创建一个文件夹
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //创建一个文件夹
        sftpUtil.createDir("/usr/local/aaa/bbb/");
        //关闭连接
        sftpUtil.close();
}
```

### 3.删除指定文件
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //删除指定文件
        sftpUtil.deleteFile("/usr/local/aaa/install.txt");
        //关闭连接
        sftpUtil.close();
}
```

### 4.把文件上传到服务器上
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //把本地的文件上传到服务器上
        FileInputStream in = new FileInputStream(new File("D:\\GoogleDown\\myvideo.mp4"));
        sftpUtil.uploadFile(in,"/usr/local/aaa/","myvideo.mp4");
        in.close();
        //关闭连接
        sftpUtil.close();
}
```

### 5.从服务器上下载文件
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //从服务器下载文件
        InputStream download = sftpUtil.downloadFile("/usr/local/aaa/", "myvideo.mp4");
        //把文件保存到本地
        File file = new File("D:\\myvideo.mp4");
        FileUtils.copyInputStreamToFile(download, file);
        download.close();
        //关闭连接
        sftpUtil.close();
}
```

### 6.执行Linux命令
(1) 调用 ls 命令打印指定目录下有哪些文件，然后返回结果
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //打印/usr/local/aaa/目录下的所有文件信息
        String result = sftpUtil.excutOrder("ls -a -l /usr/local/aaa/");
        System.out.println(result);
        //关闭连接
        sftpUtil.close();
}
```
(2) 调用Linux命令执行curl，然后返回结果
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("127.0.0.1", 22, "root", "46sdffhg");
        sftpUtil.connect();
        //执行curl 命令
        String result = sftpUtil.excutOrder("curl --location --request POST 'https://www.baidu.com'");
        System.out.println(result);
        //关闭连接
        sftpUtil.close();
}
```


### 7.综合示例
下面的示例我写了一段代码，用于将本地 Java 项目打成 jar 包，然后停止服务器上运行的 jar，删除 jar，再把本地的 jar 上传服务器启动。

```java

import com.hai.tang.util.SftpUtils;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.List;

public class MainServer {
    public static void main(String[] args) throws IOException {
        //将本地java项目打包为jar
        packageJar();
        //关闭服务器上运行的jar，删除jar,将jar包上传服务器然后运行jar
        sftpUploadRestsrt();
    }


    //本地代码打为jar包
    public static void packageJar() {
        // 项目的根目录（含有pom.xml文件）
        String projectDir = "C:\\Project\\TEST\\branches\\1.5.4.0\\my-bill\\";
        // 输出目录
        String outputDir = "C:\\Project\\TEST\\branches\\1.5.4.0\\my-bill\\lib\\";
        // 待删除的上一次生成的jar文件
        String filePath = "C:\\Project\\TEST\\branches\\1.5.4.0\\my-bill\\lib\\my-bill-release-1.5.4.0.jar";
        // 创建File对象
        File file = new File(filePath);
        // 检查文件是否存在
        if (file.exists()) {
            if (file.delete()) {
                System.out.println("文件已成功删除: " + filePath);
            } else {
                System.out.println("无法删除文件: " + filePath);
                return;
            }
        }
        // 根据项目根目录下的 pom.xml文件，分别执行 Maven 的 clean 和 package 命令打jar包
        ProcessBuilder processBuilder = new ProcessBuilder();
        processBuilder.command("C:\\Program Files\\Maven\\apache-maven-3.8.1\\bin\\mvn.cmd", "clean", "package");
        // 设置项目目录
        processBuilder.directory(new File(projectDir));
        try {
            Process process = processBuilder.start();
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
            // 等待进程结束
            int exitCode = process.waitFor();
            if (exitCode == 0) {
                System.out.println("Maven打包成功！");
            } else {
                System.out.println("Maven打包失败！");
            }
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    //jar包上传服务器并执行sh脚本重启服务，然后删除服务器的日志文件，再删除本地的日志文件
    public static void sftpUploadRestsrt() {
        System.out.println();
        //开启连接，连接自己的服务器
        SftpUtils sftpUtil = new SftpUtils("127.0.0.1", 22, "root", "test");
        try {
            sftpUtil.connect();
            //运行 service.sh 脚本 关闭服务
            String result = sftpUtil.excutOrder("sh /home/kbill/bill_temp/bill/bin/service.sh shutdown");
            if (result.contains("stop success")) {
                System.out.println("开始删除服务器jar");
                sftpUtil.excutOrder("rm -f /home/kbill/bill_temp/bill/lib/my-bill-release-1.5.4.0.jar");
                System.out.println("删除服务器jar成功" );
                //把本地的jar文件上传到服务器上
                FileInputStream in = null;
                System.out.println("开始上传jar");
                in = new FileInputStream(new File("C:\\Project\\TEST\\branches\\1.5.4.0\\my-bill\\lib\\my-bill-release-1.5.4.0.jar"));
                sftpUtil.uploadFile(in, "/home/kbill/bill_temp/bill/lib/", "my-bill-release-1.5.4.0.jar");
                in.close();
                System.out.println("上传jar成功");
                //运行 service.sh 脚本 启动服务
                String stsrtUp = sftpUtil.excutOrder("sh /home/kbill/bill_temp/bill/bin/service.sh startup");
                System.out.println("stsrtUp");
                if (stsrtUp.contains("start success")) {
                    System.out.println("启动jar成功");
                    //清空服务器上log文件里的内容
                    sftpUtil.excutOrder("> /home/kbill/bill_temp/bill/logs/kbill.log");
                    sftpUtil.excutOrder("> /home/kbill/bill_temp/bill/nohup.log");
                    System.out.println("清除日志成功");
                    //删除本地log文件
                    String filePath1 = "C:\\Users\\haitang\\Downloads\\kbill.log";
                    String filePath2 = "C:\\Users\\haitang\\Downloads\\nohup.log";
                    List<String> filePaths = Arrays.asList(filePath1, filePath2);
                    deleteFiles(filePaths);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭连接
            sftpUtil.close();
        }
    }

	//删除文件
    public static void deleteFiles(List<String> paths) {
        for (String filePath : paths) {
            // 要检查和删除的文件路径
            File file = new File(filePath);
            // 检查文件是否存在
            if (file.exists()) {
                // 尝试删除文件
                if (file.delete()) {
                    System.out.println("本地log文件已成功删除: " + filePath);
                } else {
                    System.out.println("无法删除本地log文件: " + filePath);
                }
            } else {
                System.out.println("本地log文件不存在: " + filePath);
            }
        }
    }
}
```



----




**参考**
[java连接sftp工具类](https://blog.csdn.net/u014204541/article/details/80007316)

[SringBoot中MultipartFile上传文件](https://blog.csdn.net/justry_deng/article/details/80855235)

[上传sftp,创建20171024目录,判断目录是否存在,复制文件,判断文件字符集](https://blog.csdn.net/weixin_39592763/article/details/78332503?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

[Java JSch示例在SSH Unix上运行shell命令](https://www.journaldev.com/246/jsch-example-java-ssh-unix-server)

[Java中com.jcraft.jsch.ChannelSftp讲解](https://www.cnblogs.com/weiyi1314/p/9517245.html)

[Java实现sftp及远程执行命令](https://www.jianshu.com/p/2463dc3bb0a1)

[Java Code Examples for com.jcraft.jsch.ChannelSftp.put()](https://www.programcreek.com/java-api-examples/?class=com.jcraft.jsch.ChannelSftp&method=put)

[Java Code Examples for com.jcraft.jsch.Channel.getOutputStream()](https://www.programcreek.com/java-api-examples/?class=com.jcraft.jsch.Channel&method=getOutputStream)

