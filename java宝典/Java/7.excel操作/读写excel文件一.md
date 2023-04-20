@[TOC](目录)

### 一、用到的依赖maven如下

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.2</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>

<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
</dependency>

<dependency>
    <groupId>org.apache.xmlbeans</groupId>
    <artifactId>xmlbeans</artifactId>
    <version>3.1.0</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>

<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>4.1.2</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.18</version>
</dependency>
```

### 二、excel工具类如下

```java

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.streaming.SXSSFSheet;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class ExcelUtil {

    /**
     * 读取excel
     *
     * @param file 文件路径，如：F:\\test.xls
     * @return List<List < String>>  最外层的list是每一行，内层的list是每列
     */
    public static List<List<String>> readExcle(String file) {
        Workbook workbook = null;
        try {
            workbook = WorkbookFactory.create(new FileInputStream(file));
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 这里仅取第一个sheet页签
        Sheet sheet = workbook.getSheetAt(0);
        //获取整个excel最后一列有数据的列数
        int maxColum = 0;
        for (Row row : sheet) {
            int lastRow = row.getLastCellNum();
            if (lastRow > maxColum) {
                maxColum = lastRow;
            }
        }
        //遍历
        List<List<String>> excelList = new ArrayList<>();
        for (Row row : sheet) {
            List<String> rowList = new ArrayList<>();
            for (int cn = 0; cn < maxColum; cn++) {
                Cell cell = row.getCell(cn, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
                rowList.add(cell.toString());
            }
            excelList.add(rowList);
        }
        return excelList;
    }

    /**
     * 从excel中获取指定的列
     *
     * @param file        文件路径，如：F:\\test.xls
     * @param columnIndex 要获取的列的位置，如第1列则输入：1
     * @return List<String>  返回该列的内容
     */
    public static List<String> readColum(String file, int columnIndex) {
        Workbook workbook = null;
        try {
            workbook = WorkbookFactory.create(new FileInputStream(file));
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 这里仅取第一个sheet页签
        Sheet sheet = workbook.getSheetAt(0);
        //获取整个excel最后一列有数据的列数
        int maxColum = 0;
        for (Row row : sheet) {
            int lastRow = row.getLastCellNum();
            if (lastRow > maxColum) {
                maxColum = lastRow;
            }
        }
        //遍历
        List<String> columList = new ArrayList<>();
        for (Row row : sheet) {
            Cell cell = row.getCell(columnIndex - 1, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
            columList.add(cell.toString());
        }
        return columList;
    }

    /**
     * 生成excel文件
     *
     * @param header    excel的第一行标题
     * @param date     第二行开始的数据内容
     * @param fileName  要保存的文件名 , 最终保存到 F盘
     *
     */
    public static void exportExcel(List<String> header, List<List<String>> date, String fileName) {
        SXSSFWorkbook workBook = new SXSSFWorkbook();
        //创建sheet1
        SXSSFSheet sheet = workBook.createSheet("sheet1");
        //创建第一行表头
        createHeader(header, workBook, sheet);
        //创建数据行
        createBody(date, workBook, sheet);
        //下载
        FileOutputStream out = null;
        try {
            out = new FileOutputStream("F:\\" + fileName + ".xlsx");
            workBook.write(out);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public static void createHeader(List<String> headers, Workbook workBook, Sheet sheet) {
        //sheet中创建第0行
        Row row = sheet.createRow(0);
        ///设置单元格格式为居中
        CellStyle style = workBook.createCellStyle();
        style.setAlignment(HorizontalAlignment.CENTER);
        //遍历headers插入Excel第一行做表头
        for (int i = 0; i < headers.size(); i++) {
            Cell cell = row.createCell(i);
            cell.setCellValue(headers.get(i));
            //内容居中
            cell.setCellStyle(style);
        }
    }

    public static void createBody(List<List<String>> bodies, Workbook workBook, Sheet sheet) {
        Row row;
        int rowIndex = 0;
        CellStyle style = workBook.createCellStyle();
        style.setAlignment(HorizontalAlignment.CENTER);
        for (int i = 0; i < bodies.size(); i++) {
            row = sheet.createRow(++rowIndex);
            List<String> each = bodies.get(i);
            Cell cell;
            for (int j = 0; j < each.size(); j++) {
                String cellValue = String.valueOf(each.get(j));
                cell = row.createCell(j);
                cell.setCellValue(cellValue);
                //内容居中
                cell.setCellStyle(style);
            }
        }
    }
}

```

### 三、读取文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b90d48785b14eb69a06092ca4ab4011.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6KW_5YeJ55qE5oKy5Lyk,size_11,color_FFFFFF,t_70,g_se,x_16)

```java
    public static void main(String[] args) {
        //读取excel文件为 List<List<String>> 类型
        List<List<String>> excelData = ExcelUtil.readExcle("F:\\test.xlsx");
        System.out.println("excel内容："+excelData);

        //读取excel文件第1列的数据为List<String>类型
        List<String> column = ExcelUtil.readColum("F:\\test.xlsx", 1);
        System.out.println("第一列："+column);
    }
```

输出：

```java
excel内容：[[姓名, 年龄], [张三, 18.0], [李四, 19.0], [王五, 20.0]]
第一列：[姓名, 张三, 李四, 王五]
```

### 四、生成excel文件

```java
 public static void main(String[] args) {
        //第一行表头
        List<String> headerList = Stream.of("学校名称", "年级", "班级", "姓名", "性别", "年龄").collect(Collectors.toList());
        //中间数据
        List<List<String>> bodyList = new ArrayList<>();
        bodyList.add(Arrays.asList("南湖一中", "五年级", "1班", "张三", "男", "10"));
        bodyList.add(Arrays.asList("南湖二中", "四年级", "6班", "李四", "男", "9"));
        bodyList.add(Arrays.asList("汉阳一中", "三年级", "2班", "李梅", "女", "8"));
        bodyList.add(Arrays.asList("汉口二中", "六年级", "3班", "王飞", "男", "11"));
        ExcelUtil.exportExcel(headerList, bodyList, "学生信息表");
    }
```
生成的文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b5da02743b747f780ed90236beacaa6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6KW_5YeJ55qE5oKy5Lyk,size_17,color_FFFFFF,t_70,g_se,x_16)