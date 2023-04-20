用到的jar包：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518144202853.png)

maven如下：

```xml
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.1.2</version>
        </dependency>
```

excel格式如下：
![
](https://img-blog.csdnimg.cn/20200518144346457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjk3MDk0,size_16,color_FFFFFF,t_70)

读取：

```java

import org.apache.poi.ss.usermodel.CellType;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;


public class Test {
    public static void main(String[] args) {
        try {
            //获取Excel文件
            XSSFWorkbook workbook = new XSSFWorkbook(new FileInputStream("C:\\Users\\86132\\Desktop\\aa.xlsx"));
            //获取要解析的sheet（第一个表格）
            XSSFSheet sheet = workbook.getSheetAt(0);
            //获得最后一行的行号
            int lastRowNum = sheet.getLastRowNum();
            //遍历每一行
            for (int i = 1; i <= lastRowNum; i++) {
                //获得要解析的当前行
                XSSFRow row = sheet.getRow(i);
                //将当前行的1-5列单元格设置为字符串类型
                row.getCell(0).setCellType(CellType.STRING);
                row.getCell(1).setCellType(CellType.STRING);
                row.getCell(2).setCellType(CellType.STRING);
                row.getCell(3).setCellType(CellType.STRING);
                row.getCell(4).setCellType(CellType.STRING);
                //获得当前行的1-5列单元格中的内容
               //因为单元格的类型都转化成了字符串型所有使用getStringCellValue()获取单元格内容，若单元格内容为数字类型则使用getNumericCellValue()获取数字类型单元格
               String stringCellValue0 = row.getCell(0).getStringCellValue();
                String stringCellValue1 = row.getCell(1).getStringCellValue();
                String stringCellValue2 = row.getCell(2).getStringCellValue();
                String stringCellValue3 = row.getCell(3).getStringCellValue();
                String stringCellValue4 = row.getCell(4).getStringCellValue();
                System.out.println(stringCellValue0+"--"+stringCellValue1+"--"+stringCellValue2+"--"+stringCellValue3+"--"+stringCellValue4);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```
输出如下：

```java
1--1--1--硕士及以上--硕士及以上
2--1--2--本科--本科
3--1--3--大专--大专
4--1--4--高中、中专--高中、中专
5--1--5--初中及以下--初中及以下
6--1--6--其他--其他
143--2--1--政府机关--政府机关
144--2--2--事业单位--事业单位
145--2--3--国企--国企
146--2--4--私企--私企
147--2--5--外企--外企
148--2--6--自由职业--自由职业
.......
```

**注：**
除了将一个单元格设置为字符串，有时候我们需要**将某列的单元格都设置为字符串类型（文本类型）**，否则默认为数值型当超过一定长度将会显示为科学计数法，比如身份证号，如：456587845E7。
设置第1列的单元格为字符串形式（文本形式）可如下：

```java
	  XSSFWorkbook workbook = new XSSFWorkbook(new FileInputStream("C:\\Users\\86132\\Desktop\\aa.xlsx"));
       //获取要解析的sheet（第一个表格）
       XSSFSheet sheet = workbook.getSheetAt(0);

        //设置第一列单元格格式为文本格式
        CellStyle css = workBook.createCellStyle();
        DataFormat  format = workBook.createDataFormat();
        css.setDataFormat(format.getFormat("@"));
       //列是从第0行开始的，所以第1行是0
        sheet.setDefaultColumnStyle(0,css);
```