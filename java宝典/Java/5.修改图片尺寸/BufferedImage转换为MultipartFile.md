Java里读取图片或调整图片大小可使用BufferedImage进行操作（参考我另一篇文章[修改图片大小尺寸图片缩放](修改图片大小尺寸图片缩放.md)），但有时候我们需要将BufferedImage转为MultipartFile进行其他操作可如下转换：

# 方法一：

### 1.新建ConvertToMultipartFile类实现MultipartFile接口

```java
import org.springframework.web.multipart.MultipartFile;

import java.io.*;

public class ConvertToMultipartFile implements MultipartFile {
    private byte[] fileBytes;
    String name;
    String originalFilename;
    String contentType;
    boolean isEmpty;
    long size;

    public ConvertToMultipartFile(byte[] fileBytes, String name, String originalFilename, String contentType,
                          long size) {
        this.fileBytes = fileBytes;
        this.name = name;
        this.originalFilename = originalFilename;
        this.contentType = contentType;
        this.size = size;
        this.isEmpty = false;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public String getOriginalFilename() {
        return originalFilename;
    }

    @Override
    public String getContentType() {
        return contentType;
    }

    @Override
    public boolean isEmpty() {
        return isEmpty;
    }

    @Override
    public long getSize() {
        return size;
    }

    @Override
    public byte[] getBytes() throws IOException {
        return fileBytes;
    }

    @Override
    public InputStream getInputStream() throws IOException {
        return new ByteArrayInputStream(fileBytes);
    }

    @Override
    public void transferTo(File dest) throws IOException, IllegalStateException {
        new FileOutputStream(dest).write(fileBytes);
    }
}
```
### 2.BufferedImage 转换为 MultipartFile 
BufferedImage 先转为byte[ ]，再通过上面的ConvertToMultipartFile类转为MultipartFile 
```java
	try {
            //读取图片转换为 BufferedImage
            BufferedImage image = ImageIO.read(new FileInputStream("F:/test/pic1.jpg"));
            //BufferedImage 转化为 ByteArrayOutputStream
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            ImageIO.write(image, "jpg", out);
            //ByteArrayOutputStream 转化为 byte[]
            byte[] imageByte = out.toByteArray();
            //将 byte[] 转为 MultipartFile
            MultipartFile multipartFile = new ConvertToMultipartFile(imageByte, "newNamepic", "pic1", "jpg", imageByte.length);
      } catch (IOException e) {
            e.printStackTrace();
      }
```
---
# 方法二：
引入依赖：

```java
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.3.2</version>
        <scope>compile</scope>
    </dependency>
```

```java
	try {
 			//读取图片转换为 BufferedImage
            BufferedImage image = ImageIO.read(new FileInputStream("F:/test/pic1.jpg"));
            //调整图片大小后的BufferedImage。resizeImage方法是调整图片大小的可参考文章开头我上一篇文章
            BufferedImage newImage = ImageUtils.resizeImage(image, 200, 200);
            //将newImage写入字节数组输出流
			ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ImageIO.write( newImage, "jpg", baos );
            
			//转换为MultipartFile 
            MultipartFile multipartFile = new MockMultipartFile("pic1.jpg", baos.toByteArray());
       } catch (IOException e) {
            e.printStackTrace();
       }
```

参考：
[How to convert BufferedImage to a MultiPart file without saving file to disk?](https://stackoverflow.com/questions/41163648/how-to-convert-bufferedimage-to-a-multipart-file-without-saving-file-to-disk)