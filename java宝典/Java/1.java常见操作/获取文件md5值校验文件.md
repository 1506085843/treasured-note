假如我们想校验两个文件网络传输中是否改变了，或者校验两个文件是否一致可以使用md5校验。

**代码：**

```java

import java.io.FileInputStream;
import java.io.InputStream;
import java.math.BigInteger;
import java.security.MessageDigest;

public class MainServer {
    public static void main(String[] args) {
        String filePath1 = "D:\\Download\\a.mp3";
        String filePath2 = "D:\\Download\\b.mp3";
        String file1_md5 = md5HashCode(filePath1);
        String file2_md5 = md5HashCode(filePath2);

        System.out.println(file1_md5);
        System.out.println(file2_md5);
        if(file1_md5.equals(file2_md5)){
            System.out.println("两个文件是一致的");
        }else{
            System.out.println("两个文件不一致的");
        }
     }
        
	/**
     * 获取文件md5值
     */
    public static String md5HashCode(String filePath) {
        try {
            InputStream fis = new FileInputStream(filePath);
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] buffer = new byte[1024];
            int length = -1;
            while ((length = fis.read(buffer, 0, 1024)) != -1) {
                md.update(buffer, 0, length);
            }
            fis.close();
            //转换并返回包含16个元素字节数组,返回数值范围为-128到127
            byte[] md5Bytes = md.digest();
            BigInteger bigInt = new BigInteger(1, md5Bytes);
            return bigInt.toString(16);
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }
 }
```

参考：
[Java计算文件的hash值](https://blog.csdn.net/qq_25646191/article/details/78863110)