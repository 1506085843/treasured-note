[TOC]

# 前言
有时候我们需要让从代码里远程连接服务器进行文件上传、下载、判断文件路径是否存在、创建文件夹等操作。

这时候就用到了sftp。常见的三个库是：JSch、SSHJ 和 Apache Commons VFS它们都能实现远程连接服务器。

本文主要讲解利用JSch远程连接服务器。


# 一、所需依赖
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

# 二、工具类

SftpUtil 工具类如下：
```java

 
 import com.jcraft.jsch.*;
 import org.apache.commons.io.IOUtils;

 import java.io.IOException;
 import java.io.InputStream;
 import java.nio.charset.Charset;
 import java.util.Properties;

public class SftpUtil {
    private static JSch jsch;
    private static Session session = null;
    private static Channel channel = null;
    private static ChannelSftp channelSftp = null;

    //服务器用户名
    private String ftpUserName;

    //服务器密码
    private String ftpPassword;

    //服务器ip
    private String ftpHost;

    //服务器端口
    private String ftpPort;

    public SftpUtil() {
    }

    public SftpUtil(String ftpUserName, String ftpPassword, String ftpHost, String ftpPort) {
        this.ftpUserName = ftpUserName;
        this.ftpPassword = ftpPassword;
        this.ftpHost = ftpHost;
        this.ftpPort = ftpPort;
    }

    /**
     * 开启连接
     */
    public ChannelSftp connect() {
        jsch = new JSch();
        try {
            // 根据用户名、主机ip、端口号获取一个Session对象
            session = jsch.getSession(ftpUserName, ftpHost, Integer.valueOf(ftpPort));
            // 设置密码
            session.setPassword(ftpPassword);
            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            // 为Session对象设置properties
            session.setConfig(config);
            // 设置连接超时为5秒
            session.setTimeout(100 * 50);
            // 通过Session建立连接
            session.connect();
            // 打开SFTP通道
            channel = session.openChannel("sftp");
            // 建立SFTP通道的连接
            channel.connect();
            channelSftp = (ChannelSftp) channel;
        } catch (JSchException e) {
            e.printStackTrace();
        }
        return channelSftp;
    }

    /**
     * 关闭连接
     */
    public void close() {
        if (channel != null) {
            channel.disconnect();
        }
        if (session != null) {
            session.disconnect();
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
            SftpATTRS sftpATTRS = channelSftp.lstat(directory);
            dirExist = sftpATTRS.isDir();
        } catch (Exception e) {
            if (e.getMessage().equalsIgnoreCase("no such file")) {
                dirExist = false;
            }
        }
        return dirExist;
    }

    /**
     * 创建一个文件夹(若整个路径都不存在会依次创建，若改路径已经存在则不会创建)
     *
     * @param createpath 要创建的文件夹路径，如：/root/test/saveFile/
     * @throws SftpException
     */
    public void createDir(String createpath) {
        createpath = null != createpath && createpath.endsWith("/") ? createpath : createpath + "/";
        if (!isDirExist(createpath)) {
            StringBuilder builder = new StringBuilder("/");
            String pathArry[] = createpath.split("/");
            for (String dir : pathArry) {
                if (!dir.equals("")) {
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
        } catch (JSchException e) {
            e.printStackTrace();
        } catch (IOException e) {
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

# 三、测试
## 1.判断指定目录是否存在
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
        sftpUtil.connect();
        //判断目录是否存在
        boolean dirBoolean = sftpUtil.isDirExist("/usr/local/aaa/");
        //关闭连接
        sftpUtil.close();
}
```


## 2.创建一个文件夹
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
        sftpUtil.connect();
        //创建一个文件夹
        sftpUtil.createDir("/usr/local/aaa/bbb/");
        //关闭连接
        sftpUtil.close();
}
```

## 3.删除指定文件
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
        sftpUtil.connect();
        //删除指定文件
        sftpUtil.deleteFile("/usr/local/aaa/install.txt");
        //关闭连接
        sftpUtil.close();
}
```

## 4.把文件上传到服务器上
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
        sftpUtil.connect();
        //把本地的文件上传到服务器上
        FileInputStream in = new FileInputStream(new File("D:\\GoogleDown\\myvideo.mp4"));
        sftpUtil.uploadFile(in,"/usr/local/aaa/","myvideo.mp4");
        in.close();
        //关闭连接
        sftpUtil.close();
}
```

## 5.从服务器上下载文件
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
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

## 6.执行Linux命令
(1) 调用 ls 命令打印指定目录下有哪些文件，然后返回结果
```java
public static void main(String[] args) throws IOException {
		//开启连接
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
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
        SftpUtil sftpUtil = new SftpUtil("root", "46sdffhg", "127.0.0.1", "22");
        sftpUtil.connect();
        //打印/usr/local/aaa/目录下的所有文件信息
        String result = sftpUtil.excutOrder("curl --location --request POST 'https://www.baidu.com'");
        System.out.println(result);
        //关闭连接
        sftpUtil.close();
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