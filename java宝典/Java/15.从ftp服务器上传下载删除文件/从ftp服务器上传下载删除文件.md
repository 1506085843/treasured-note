  

# 一、maven依赖
```xml
        <dependency>
            <groupId>commons-net</groupId>
            <artifactId>commons-net</artifactId>
            <version>3.11.1</version>
        </dependency>
```


# 二、工具类

```java

import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;
import org.apache.commons.net.ftp.FTPFile;
import org.apache.commons.net.ftp.FTPReply;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class FtpUtils {
    private static FTPClient ftpClient = null;

    //服务器用户名
    private String user;

    //服务器密码
    private String password;

    //服务器ip
    private String host;

    //服务器端口
    private int port;

    public FtpUtils() {
    }

    public FtpUtils(String host, int port, String user, String password) {
        this.user = user;
        this.password = password;
        this.host = host;
        this.port = port;
    }

    /**
     * 开启连接并登录
     */
    public FTPClient connect() {
        ftpClient = new FTPClient();
        try {
            ftpClient.connect(host, port);
            int reply = ftpClient.getReplyCode();
            if (!FTPReply.isPositiveCompletion(reply)) {
                ftpClient.disconnect();
                throw new IOException("FTP server refused connection.");
            }
            if (!ftpClient.login(user, password)) {
                ftpClient.logout();
                throw new IOException("Failed to login to FTP server.");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return ftpClient;
    }

    /**
     * 关闭连接
     */
    public void close() {
        try {
            if (ftpClient != null && ftpClient.isConnected()) {
                ftpClient.logout();
                ftpClient.disconnect();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    /**
     * 查询文件夹下所有文件
     *
     * @param dirPath 要查询的文件夹路径，如：/root/local/data/
     */
    public List<String> filesList(String dirPath) {
        FTPFile[] files = new FTPFile[0];
        try {
            ftpClient.changeWorkingDirectory(dirPath);
            files = ftpClient.listFiles(dirPath);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return Arrays.stream(files).map(FTPFile::getName).collect(Collectors.toList());
    }

    /**
     * 下载文件 (必须确保 FTP 服务器给登录用户提供了"读取"权限)
     *
     * @param source      要下载的文件路径，如：/root/local/data/test.sql
     * @param destination 下载到本地的文件路径，如：C:\Users\haitang\Downloads\downloaded_buz.sql
     */
    public void downloadFile(String source, String destination) {
        try(FileOutputStream out = new FileOutputStream(destination)) {
            ftpClient.retrieveFile(source, out);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 上传文件
     *
     * @param path 要上传到的文件路径及名称，如：/root/local/data/test.sql
     * @param file 要上传到的文件
     */
    public void uploadFile(String path, File file) {
        try (FileInputStream in = new FileInputStream(file)) {
            ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
            ftpClient.storeFile(path, in);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 删除文件(必须确保 FTP 服务器给登录用户提供了"删除"权限)
     *
     * @param filePath 要上传到的文件路径及名称，如：/root/local/data/test.sql
     */
    public void deleteFile(String filePath) {
        try {
            ftpClient.deleteFile(filePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 新建文件夹
     *
     * @param dirPath 要新建的文件夹名称
     * @return 返回创建文件夹是否成功
     */
    public boolean createDir(String dirPath) {
        try {
            String[] dirs = dirPath.split("/");
            ftpClient.changeWorkingDirectory("/");
            StringBuilder path = new StringBuilder("/");
            for (String dir : dirs) {
                if (dir == null || dir.isEmpty()) {
                    continue;
                }
                path.append(dir).append("/");
                if (!ftpClient.changeWorkingDirectory(path.toString())) {
                    if (!ftpClient.makeDirectory(path.toString())) {
                        return false;
                    }
                    ftpClient.changeWorkingDirectory(path.toString());
                }
            }
            return true;
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }
    }
    
}


```

# 三、测试
## 1.获取文件夹下的所有文件名

```java
      FtpUtils  ftpClient = new FtpUtils("127.0.0.1", 21, "root", "123qaz");
	  ftpClient.connect();
	  //获取 /root/test/ 下的所有文件名
	  List<String> files = ftpClient.filesList("/root/test/");
	  ftpClient.close();
```
## 2.下载文件

```java
      FtpUtils  ftpClient = new FtpUtils("127.0.0.1", 21, "root", "123qaz");
	  ftpClient.connect();
	  //下载 "/root/local/data/my.sql" 到本地 "C:\\Users\\haitang\\Downloads\\downloaded.sql"
	  ftpClient.downloadFile("/root/local/data/my.sql", "C:\\Users\\haitang\\Downloads\\downloaded.sql");
	  ftpClient.close();
```
## 3.上传文件

```java
      FtpUtils  ftpClient = new FtpUtils("127.0.0.1", 21, "root", "123qaz");
	  ftpClient.connect();
	  //上传 "C:\\Users\\haitang\\Downloads\\qaz.sql" 到 "/root/local/test/buz.sql"
	  File file = new File("C:\\Users\\haitang\\Downloads\\qaz.sql");
      ftpClient.uploadFile("/root/local/test/buz.sql",file );
	  ftpClient.close();
```

## 4.删除文件

```java
      FtpUtils  ftpClient = new FtpUtils("127.0.0.1", 21, "root", "123qaz");
	  ftpClient.connect();
	  //删除 "/root/local/data/test.txt"
	  ftpClient.deleteFile("/root/local/data/test.txt" );
	  ftpClient.close();
```

## 5.创建文件夹

```java
      FtpUtils  ftpClient = new FtpUtils("127.0.0.1", 21, "root", "123qaz");
	  ftpClient.connect();
	  //创建文件夹 "/test/local/java/"
	  ftpClient.createDir("/test/local/java/" );
	  ftpClient.close();
```