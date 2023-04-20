[TOC]
# 一、工具类
工具类ZipUtils 如下：

```java
package utils;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

public class ZipUtils {
    /**
     *
     * 文件夹压缩成ZIP
     *
     * @param srcDir           压缩文件夹路径
     * @param out              压缩文件输出流
     * @param keepDirStructure 是否保留原来的目录结构,true:保留目录结构;
     *
     *                         false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
     *
     * @throws RuntimeException 压缩失败会抛出运行时异常
     *
     */
    public static void toZip(String srcDir, OutputStream out, boolean keepDirStructure)
            throws RuntimeException {
        long start = System.currentTimeMillis();
        ZipOutputStream zos = null;
        try {
            zos = new ZipOutputStream(out);
            File sourceFile = new File(srcDir);
            compress(sourceFile, zos, sourceFile.getName(), keepDirStructure);
            long end = System.currentTimeMillis();
            System.out.println("压缩完成，耗时：" + (end - start) + " ms");
        } catch (Exception e) {
            throw new RuntimeException("zip error from ZipUtils", e);
        } finally {
            if (zos != null) {
                try {
                    zos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    /**
     * 多文件压缩成ZIP
     *
     * @param imageMap 需要压缩的文件列表，键值对为 <文件名，文件的字节数组>，文件名必须包含后缀
     * @param out      压缩文件输出流
     * @throws RuntimeException 压缩失败会抛出运行时异常
     */
    public static void toZip(Map<String, byte[]> imageMap, OutputStream out) throws RuntimeException {
        long start = System.currentTimeMillis();
        try (ZipOutputStream zos = new ZipOutputStream(out)) {
            for (Map.Entry<String, byte[]> map : imageMap.entrySet()) {
                zos.putNextEntry(new ZipEntry(map.getKey()));
                zos.write(map.getValue());
                zos.closeEntry();
            }
            long end = System.currentTimeMillis();
            System.out.println("压缩完成，耗时：" + (end - start) + " ms");
        } catch (Exception e) {
            throw new RuntimeException("zip error from ZipUtils", e);
        }
    }


    /**
     * 多文件压缩成ZIP
     * @param srcFiles 需要压缩的文件列表
     * @param out           压缩文件输出流
     * @throws RuntimeException 压缩失败会抛出运行时异常
     */
    public static void toZip(List<File> srcFiles , OutputStream out)throws RuntimeException {
        long start = System.currentTimeMillis();
        try(ZipOutputStream zos= new ZipOutputStream(out)) {
            for (File srcFile : srcFiles) {
                byte[] buf = new byte[2048];
                zos.putNextEntry(new ZipEntry(srcFile.getName()));
                int len;
                FileInputStream in = new FileInputStream(srcFile);
                while ((len = in.read(buf)) != -1){
                    zos.write(buf, 0, len);
                }
                zos.closeEntry();
                in.close();
            }
            long end = System.currentTimeMillis();
            System.out.println("压缩完成，耗时：" + (end - start) +" ms");
        } catch (Exception e) {
            throw new RuntimeException("zip error from ZipUtils",e);
        }
    }



    /**
     *
     * 递归压缩方法
     * @param sourceFile       源文件
     * @param zos              zip输出流
     * @param name             压缩后的名称
     * @param keepDirStructure 是否保留原来的目录结构,true:保留目录结构;
     *
     *                         false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
     * @throws Exception
     */
    private static void compress(File sourceFile, ZipOutputStream zos, String name, boolean keepDirStructure) throws Exception {
        byte[] buf = new byte[2048];
        if (sourceFile.isFile()) {
            // 向zip输出流中添加一个zip实体，构造器中name为zip实体的文件的名字
            zos.putNextEntry(new ZipEntry(name));
            // copy文件到zip输出流中
            int len;
            FileInputStream in = new FileInputStream(sourceFile);
            while ((len = in.read(buf)) != -1) {
                zos.write(buf, 0, len);
            }
            // Complete the entry
            zos.closeEntry();
            in.close();
        } else {
            File[] listFiles = sourceFile.listFiles();
            if (listFiles == null || listFiles.length == 0) {
                // 需要保留原来的文件结构时,需要对空文件夹进行处理
                if (keepDirStructure) {
                    // 空文件夹的处理
                    zos.putNextEntry(new ZipEntry(name + "/"));
                    // 没有文件，不需要文件的copy
                    zos.closeEntry();
                }
            } else {
                for (File file : listFiles) {
                    // 判断是否需要保留原来的文件结构
                    if (keepDirStructure) {
                        // 注意：file.getName()前面需要带上父文件夹的名字加一斜杠,
                        // 不然最后压缩包中就不能保留原来的文件结构,即：所有文件都跑到压缩包根目录下了
                        compress(file, zos, name + "/" + file.getName(), keepDirStructure);
                    } else {
                        compress(file, zos, file.getName(), keepDirStructure);
                    }
                }
            }
        }
    }

}

```

# 二、 测试
## 1.压缩文件夹

```java
import utils.ZipUtils;
import java.io.*;

public class Test {
    public static void main(String[] args) {
   		 try {
           		 //定义输出流和最终生成的文件名，如;保存到 F盘的 MyPic.zip
            	 FileOutputStream   fileOutputStream = new FileOutputStream(new File("F:/Myfolder.zip"));
           		 BufferedOutputStream  bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
            	//开始压缩 F盘下的test文件夹
            	ZipUtils.toZip("F:/test", bufferedOutputStream,true);
       		 } catch (FileNotFoundException e) {
           			 e.printStackTrace();
       		 }
     }
}
```

## 2 .压缩多文件

```java
import java.io.*;
import java.util.ArrayList;
import utils.ZipUtils;
import java.util.List;

public class Test {
    public static void main(String[] args) {
   		try {
   			//要压缩的文件列表
            List<File> fileList = new ArrayList<>();
            fileList.add(new File("F:/test/pic1.jpg"));
            fileList.add(new File("F:/test/doc_20210203100237.pdf"));
            //压缩后的文件名和保存路径
            FileOutputStream fos2 = new FileOutputStream(new File("f:/test/Myzip.zip"));
            //开始压缩
            ZipUtils.toZip(fileList, fos2);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
     }
}
```
## 3.有时候我们可能只有文件流或者文件的字节数组，可通过字文件节数组压缩

需要导入Apache Commons IO包利用 IOUtils.toByteArray 将 InputStream 转换为 byte[]

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency>
```


```java
import org.apache.commons.io.IOUtils;
import utils.ZipUtils;
import java.io.*;
import java.util.HashMap;
import java.util.Map;

public class Test {
    public static void main(String[] args) {
        try {
            //读取图片转换为byte[]
            FileInputStream inputStream1 = new FileInputStream("F:/test/pic1.jpg");
            byte[] bytes1 = IOUtils.toByteArray(inputStream1);
            //读取pdf转换为byte[]
            FileInputStream inputStream2 = new FileInputStream("F:/test/doc_20210203100237.pdf");
            byte[] bytes2 = IOUtils.toByteArray(inputStream2);
            //读取txt转换为byte[]
            FileInputStream inputStream3 = new FileInputStream("F:/test/za.txt");
            byte[] bytes3 = IOUtils.toByteArray(inputStream3);

            //三个文件存入picMap
            Map<String, byte[]> picMap = new HashMap<>();
            picMap.put("pic1.jpg", bytes1);
            picMap.put("doc_20210203100237.pdf", bytes2);
            picMap.put("za.txt", bytes3);

            //定义输出流和最终生成的文件名，如;MyPic.zip
            FileOutputStream fileOutputStream = new FileOutputStream(new File("F:/test/MyPic.zip"));
            BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
            //开始压缩
            ZipUtils.toZip(picMap, bufferedOutputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

---

参考：
[Convert InputStream to byte array in Java](https://stackoverflow.com/questions/1264709/convert-inputstream-to-byte-array-in-java)
[Java实现将文件或者文件夹压缩成zip](https://www.cnblogs.com/zeng1994/p/7862288.html)