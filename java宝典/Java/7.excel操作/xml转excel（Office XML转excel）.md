[TOC]

# 前言
旧版的 excel 文件 Office XML是xml类型的，也称为SpreadsheetML类型，很古老的excel类型了是2002年左右的格式，现在的格式都是用的 xls 或者 xlsx。遇到的问题就是要把xml类型的 Office XML转化为 xlsx的excel，所以写了本篇文章方便以后遇到这个问题的人。

xml转化为excel可以采用安装JODConverter+OpenOffice ，然后使用JODConverter来将xml转化为excel，例如：

```java
  public static void main(String[] args) {
    LocalOfficeManager officeManager = LocalOfficeManager.install();
    try {
      officeManager.start();
      File source = new File("C:\\tmp\\jodc\\2019_enlarged.xml");
      File target = new File("C:\\tmp\\jodc\\2019_enlarged.xlsx");
      LocalConverter.builder().build().convert(source).to(target).execute();
    } catch (Exception e) {
      OfficeUtils.stopQuietly(officeManager);
    }
  }
```

**只不过OpenOffice安装包100多M太大了，所以本文通过另一种方式手动解析xml来转换为excel文件
（注意，本文的代码只支持横向的单元格合并，纵向合并的单元格因为会受到横向合并的单元格而对不齐）**

# 一、maven如下

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

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.83</version>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.8.0</version>
        </dependency>
```

# 二、解析类
## SAXHandler类

```java
package com.diff;


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;

import java.util.ArrayList;
import java.util.List;

public class SAXHandler extends DefaultHandler {

    private List<XmlRow> xmlRowList = new ArrayList<>();
    //存储所有Worksheet的内容
    public List<List<XmlRow>> xmlSheetList = new ArrayList<>();
    //存储所有Worksheet的ss:Name属性值
    public List<String> sheetNameList = new ArrayList<>();


    XmlRow xmlRow = null;
    String content = null;
    Integer mergeAcross = null;
    Integer mergeDown = null;

    @Override
    //当开始标签被找到时
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        int size = attributes.getLength();
        switch (qName) {
            //Create a new Row object when the start tag is found
            case "Worksheet":
                xmlRowList.clear();
                for (int i = 0; i < size; i++) {
                    String attName = attributes.getQName(i);
                    if ("ss:Name".equals(attName)) {
                        sheetNameList.add(attributes.getValue(i));
                    }
                }
                break;
            case "Row":
                xmlRow = new XmlRow();
                break;
            case "Cell":
                mergeAcross = mergeDown = 0;
                for (int i = 0; i < size; i++) {
                    String attName = attributes.getQName(i);
                    if ("ss:MergeAcross".equals(attName)) {
                        mergeAcross = Integer.parseInt(attributes.getValue(i));
                    }
                    if ("ss:MergeDown".equals(attName)) {
                        mergeDown = Integer.parseInt(attributes.getValue(i));
                    }
                }
                break;
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        switch (qName) {
            case "Worksheet":
                //xmlRowList深拷贝为newXmlRowList
                String listStr = JSONObject.toJSONString(xmlRowList);
                List<XmlRow> newXmlRowList = JSON.parseArray(listStr, XmlRow.class);
                xmlSheetList.add(newXmlRowList);
                break;
            case "Row":
                xmlRowList.add(xmlRow);
                break;
            case "Cell":
                xmlRow.cellMergeAcrossList.add(mergeAcross);
                xmlRow.cellMergeDownList.add(mergeDown);
                break;
            case "Data":
                xmlRow.cellList.add(content);
                break;
        }
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        content = String.copyValueOf(ch, start, length).trim();
    }
}

```
## XmlConvertExcel类

```java
package com.diff;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import org.apache.commons.io.IOUtils;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.xml.sax.SAXException;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.*;
import java.nio.charset.Charset;
import java.util.List;

public class XmlConvertExcel {

    public Workbook workbook = null;
    public Sheet sheet = null;
    public CellStyle style = null;

    public XmlConvertExcel() {
        workbook = new XSSFWorkbook();
        style = workbook.createCellStyle();
        style.setAlignment(HorizontalAlignment.CENTER);
        style.setVerticalAlignment(VerticalAlignment.CENTER);
    }

    /**
     * 解析xml数据 返回 handler
     */
    public SAXHandler parseXmlData(String xmlPath) throws ParserConfigurationException, SAXException, IOException {
        //解析xml文件内容为 handler.xmlRowList ,xmlRowList中包含了单元格内容，横向合并单元格数量和纵向合并单元格数量
        File file = new File(xmlPath);
        SAXParserFactory parserFactor = SAXParserFactory.newInstance();
        SAXParser parser = parserFactor.newSAXParser();
        SAXHandler handler = new SAXHandler();
        String fileContent = IOUtils.toString(new FileInputStream(file), Charset.defaultCharset());
        ByteArrayInputStream bis = new ByteArrayInputStream(fileContent.getBytes());
        parser.parse(bis, handler);
        return handler;
    }


    /**
     * xml转excel类型文件
     *
     * @param xmlPath      xml路径 ，如：F:\excels\bbb.xml
     * @param outExcelPath 最终生成的excel路径 ，如：F:\excels\fileOut.xlsx
     */
    public void xmlToExcel(String xmlPath, String outExcelPath) throws IOException, ParserConfigurationException, SAXException {
        //解析xml获得具体数据
        SAXHandler handler = parseXmlData(xmlPath);
        handleSheet(handler);
        //数据写入到最终xlsx文件
        FileOutputStream fout = new FileOutputStream(outExcelPath);
        workbook.write(fout);
        workbook.close();
        fout.close();
    }

    public void handleSheet(SAXHandler handler) {
        //深拷贝xmlSheetList 为 newXmlSheetList .
        String listStr = JSONObject.toJSONString(handler.xmlSheetList);
        List<List<XmlRow>> newXmlSheetList = JSON.parseObject(listStr, new TypeReference<List<List<XmlRow>>>() {
        });
        int sheetIndex = 0;
        for (List<XmlRow> xmlRowList : newXmlSheetList) {
            String sheetName = handler.sheetNameList.get(sheetIndex);
            sheet = workbook.createSheet(sheetName);
            handleRow(xmlRowList);
            sheetIndex++;
        }
    }

    public void handleRow(List<XmlRow> xmlRowList) {
        //深拷贝xmlSheetList 为 newXmlSheetList .
        String listStr = JSONObject.toJSONString(xmlRowList);
        List<XmlRow> newXmlRowList = JSON.parseObject(listStr, new TypeReference<List<XmlRow>>() {
        });

        //遍历每一行处理要合并的单元格（即如果该单元格是要合并n个单元格就在其后插入n个空内容）
        for (int i = 0; i < xmlRowList.size(); i++) {
            //每一行的数据包括单元格内容、横向要合并的单元格数量、纵向合并的单元格数量
            XmlRow subsRow =  xmlRowList.get(i);
            List<Integer> mergeAcrossList = subsRow.cellMergeAcrossList;
            List<Integer> mergeDownList = subsRow.cellMergeDownList;
            List<String> cellList = subsRow.cellList;

            //拷贝后的每一行的数据包括单元格内容、横向要合并的单元格数量、纵向合并的单元格数量
            XmlRow newSubsRow = newXmlRowList.get(i);
            //该行每个单元格的内容
            List<String> newCellList = newSubsRow.cellList;
            //该行每个单元格的横向合并单元格的数量
            List<Integer> newMergeAcrossList = newSubsRow.cellMergeAcrossList;
            //该行每个单元格的纵向合并单元格的数量
            List<Integer> newMergeDownList = newSubsRow.cellMergeDownList;

            int cellCount = 0;
            int addSum = 0;
            //遍历该行单元格
            for (int j = 0; j < cellList.size(); j++) {
                String cellValue = cellList.get(j);//该单元格内容
                Integer cellMergeAcross = mergeAcrossList.get(j);//该单元格横向合并单元格数量
                Integer cellMergeDown = mergeDownList.get(j);//该单元格纵向合并单元格数量
                if (cellMergeAcross != 0) {
                    Integer temp = cellCount + addSum;
                    for (int n = 0; n < cellMergeAcross; n++) {
                        temp = temp + 1;
                        //该单元格后要合并的单元格填充为“”
                        newCellList.add(temp, "");
                        addSum++;
                    }
                }
                if (cellMergeDown != 0) {
                    for (int n = 0; n < cellMergeDown; n++) {
                        //修改原数据的下一行数据的横合并数据
                        XmlRow nextRow = xmlRowList.get(i + n + 1);
                        List<Integer> nextMergeAcrossList = nextRow.cellMergeAcrossList;
                        nextMergeAcrossList.set(j, cellMergeAcross);
                    }
                }
                cellCount++;
            }
        }
        writeCell(xmlRowList, newXmlRowList);
    }

    public void writeCell(List<XmlRow> xmlRowList, List<XmlRow> newXmlRowList) {
        //数据写入单元格
        int rowCount = 0;
        for (XmlRow subsRow : newXmlRowList) {
            Row row = sheet.createRow(rowCount);
            int cellCount = 0;
            for (String cellValue : subsRow.cellList) {
                Cell cell = row.createCell(cellCount);
                cell.setCellValue(cellValue);
                cell.setCellStyle(style);
                cellCount++;
            }
            rowCount++;
        }

        //合并单元格
        rowCount = 0;
        int rowMerge = 0;
        for (XmlRow subsRow : xmlRowList) {
            int cellCount = 0;
            for (String cellValue : subsRow.cellList) {
                Integer mergeAcross = subsRow.cellMergeAcrossList.get(cellCount);
                Integer mergeDown = subsRow.cellMergeDownList.get(cellCount);
                if (mergeAcross != 0 || mergeDown != 0) {
                    int firstCoumn = rowMerge == 0 ? cellCount + mergeDown : cellCount + rowMerge;
                    int endCoumn = rowMerge == 0 ? cellCount + mergeAcross : cellCount + rowMerge + mergeAcross;
                    //不支持合并纵向单元格
                    if (endCoumn >= firstCoumn) {
                        CellRangeAddress region = new CellRangeAddress(rowCount, rowCount + mergeDown, firstCoumn, endCoumn);
                        sheet.addMergedRegion(region);
                    }

                    if (mergeAcross != 0) {
                        rowMerge += mergeAcross;
                    }
                }
                cellCount++;
            }
            rowMerge = 0;
            rowCount++;
        }
    }
}

```
## XmlRow类

```java
package com.diff;

import java.util.ArrayList;

public class XmlRow {
    //存储Cell中Data的数据
    public ArrayList<String> cellList = new ArrayList<>();
    //存储Cell中ss:MergeAcross的属性值
    public ArrayList<Integer> cellMergeAcrossList = new ArrayList<>();
    //存储Cell中ss:MergeDown的属性值
    public ArrayList<Integer> cellMergeDownList = new ArrayList<>();

    @Override
    public String toString() {
        return cellList.toString();
    }
}
```

# 三、测试
## 测试用到的bbb.xml

```xml
<?xml version="1.0"?>
<?mso-application progid="Excel.Sheet"?>
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet"
 xmlns:o="urn:schemas-microsoft-com:office:office"
 xmlns:x="urn:schemas-microsoft-com:office:excel"
 xmlns:dt="uuid:C2F41010-65B3-11d1-A29F-00AA00C14882"
 xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet"
 xmlns:html="http://www.w3.org/TR/REC-html40">
 <DocumentProperties xmlns="urn:schemas-microsoft-com:office:office">
  <LastAuthor>测试</LastAuthor>
  <LastPrinted>2014-01-06T07:22:14Z</LastPrinted>
  <Created>1996-12-17T01:32:42Z</Created>
  <LastSaved>2022-05-19T06:24:50Z</LastSaved>
  <Version>16.00</Version>
 </DocumentProperties>
 <CustomDocumentProperties xmlns="urn:schemas-microsoft-com:office:office">
  <KSOProductBuildVer dt:dt="string">2052-11.1.0.11691</KSOProductBuildVer>
  <ICV dt:dt="string">CEC1577D7A0F4685BB4FC1A44A4A2CCF</ICV>
 </CustomDocumentProperties>
 <OfficeDocumentSettings xmlns="urn:schemas-microsoft-com:office:office">
  <AllowPNG/>
 </OfficeDocumentSettings>
 <ExcelWorkbook xmlns="urn:schemas-microsoft-com:office:excel">
  <WindowHeight>12165</WindowHeight>
  <WindowWidth>28800</WindowWidth>
  <WindowTopX>32767</WindowTopX>
  <WindowTopY>32767</WindowTopY>
  <ProtectStructure>False</ProtectStructure>
  <ProtectWindows>False</ProtectWindows>
 </ExcelWorkbook>
 <Styles>
  <Style ss:ID="Default" ss:Name="Normal">
   <Alignment ss:Vertical="Bottom"/>
   <Borders/>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
   <Interior/>
   <NumberFormat/>
   <Protection/>
  </Style>
  <Style ss:ID="m1392413137156">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
  </Style>
  <Style ss:ID="m1392413136324">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
  </Style>
  <Style ss:ID="m1392413136364">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
  </Style>
  <Style ss:ID="m1392413136404">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
  </Style>
  <Style ss:ID="s62">
   <Alignment ss:Vertical="Bottom" ss:ShrinkToFit="1"/>
   <Borders/>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
   <Interior/>
   <NumberFormat/>
   <Protection/>
  </Style>
  <Style ss:ID="s64">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders/>
   <Font ss:FontName="黑体" x:CharSet="134" x:Family="Modern" ss:Size="18"
    ss:Bold="1"/>
  </Style>
  <Style ss:ID="s65">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12" ss:Bold="1"/>
  </Style>
  <Style ss:ID="s66">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="10.5" ss:Bold="1"/>
  </Style>
  <Style ss:ID="s67">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
  </Style>
  <Style ss:ID="s68">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
  </Style>
  <Style ss:ID="s69">
   <Alignment ss:Horizontal="Center" ss:Vertical="Center" ss:ShrinkToFit="1"/>
   <Borders>
    <Border ss:Position="Bottom" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Left" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Right" ss:LineStyle="Continuous" ss:Weight="1"/>
    <Border ss:Position="Top" ss:LineStyle="Continuous" ss:Weight="1"/>
   </Borders>
   <Font ss:FontName="宋体" x:CharSet="134"/>
   <NumberFormat ss:Format="###0.00_ "/>
  </Style>
 </Styles>
 <Worksheet ss:Name="薪酬发放表">
  <Table ss:ExpandedColumnCount="19" ss:ExpandedRowCount="32" x:FullColumns="1"
   x:FullRows="1" ss:StyleID="s62" ss:DefaultColumnWidth="54"
   ss:DefaultRowHeight="14.25">
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="73.5"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="33.75"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="63" ss:Span="1"/>
   <Column ss:Index="5" ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="74.25"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="66"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="79.5"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="66" ss:Span="2"/>
   <Column ss:Index="11" ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="76.5"/>
   <Column ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="66" ss:Span="4"/>
   <Column ss:Index="17" ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="76.5"
    ss:Span="1"/>
   <Column ss:Index="19" ss:StyleID="s62" ss:AutoFitWidth="0" ss:Width="100.5"/>
   <Row ss:AutoFitHeight="0" ss:Height="33.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"><Data ss:Type="String">2022年1月xxx公司托管人员五险一金明细表</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:StyleID="s65"><Data ss:Type="String">部门</Data></Cell>
    <Cell ss:StyleID="s65"><Data ss:Type="String">序号</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果组织名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果职位名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">工伤单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗补充保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">单位社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">备注</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68" ss:Formula="=IF(RC[1]=&quot;&quot;,&quot;&quot;,COUNTA(R3C3:RC[1]))"><Data
      ss:Type="Number">1</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">张三</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心负责人</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1252.43</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">313.11</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">78.28</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1920</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">3563.82</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">2504.86</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1252.43</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">78.28</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">62.62</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">469.66</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1920</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">6287.85</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">3563.82</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68" ss:Formula="=IF(RC[1]=&quot;&quot;,&quot;&quot;,COUNTA(R3C3:RC[1]))"><Data
      ss:Type="Number">2</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">李四</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心系统运维专员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">326.67</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">81.67</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">20.420000000000002</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">299</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">727.76</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">653.33000000000004</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">326.67</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">20.420000000000002</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">16.329999999999998</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">122.5</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">299</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1438.25</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">727.76</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68" ss:Formula="=IF(RC[1]=&quot;&quot;,&quot;&quot;,COUNTA(R3C3:RC[1]))"><Data
      ss:Type="Number">3</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">王五</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">信息中心系统运维专员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">704.11</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">176.03</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">44.01</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1029</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1953.15</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1408.23</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">704.11</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">44.01</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">35.21</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">264.04000000000002</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1029</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">3484.6</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1953.15</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:MergeAcross="4" ss:StyleID="m1392413136324"><Data ss:Type="String">小计：</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2283.21</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">570.81</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">142.71</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">3248.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">6244.73</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">4566.42</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2283.21</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">142.71</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">114.16</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">856.20</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">3248.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">11210.70</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">6244.73</Data></Cell>
    <Cell ss:StyleID="s68"/>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21"/>
   <Row ss:AutoFitHeight="0" ss:Height="33.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"><Data ss:Type="String">2022年2月xxx公司托管人员五险一金明细表</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:StyleID="s65"><Data ss:Type="String">部门</Data></Cell>
    <Cell ss:StyleID="s65"><Data ss:Type="String">序号</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果组织名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果职位名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">工伤单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗补充保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">单位社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">备注</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">公租房管理中心</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">4</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">张三</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">公租房管理中心</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">居住区整理公司公租房管理中心综合专员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">688.24</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">172.06</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">43.02</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1904.32</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1376.49</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">688.24</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">43.02</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">34.409999999999997</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">258.08999999999997</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">3401.25</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1904.32</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:MergeAcross="4" ss:StyleID="m1392413136364"><Data ss:Type="String">小计：</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">688.24</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">172.06</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">43.02</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1001.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1904.32</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1376.49</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">688.24</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">43.02</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">34.41</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">258.09</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1001.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">3401.25</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1904.32</Data></Cell>
    <Cell ss:StyleID="s68"/>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21"/>
   <Row ss:AutoFitHeight="0" ss:Height="33.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"><Data ss:Type="String">2022年3月xxx公司托管人员五险一金明细表</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:StyleID="s65"><Data ss:Type="String">部门</Data></Cell>
    <Cell ss:StyleID="s65"><Data ss:Type="String">序号</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果组织名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果职位名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">工伤单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗补充保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">单位社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">备注</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">城投办公楼</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">5</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">小风</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">城投办公楼</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">城安物业公司城投办公楼项目主管</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">807.58</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">201.9</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">50.47</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1249</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">2308.9499999999998</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1615.16</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">807.58</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">50.47</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">40.380000000000003</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">302.83999999999997</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1249</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">4065.43</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">2308.9499999999998</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:MergeAcross="4" ss:StyleID="m1392413136404"><Data ss:Type="String">小计：</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">807.58</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">201.90</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">50.47</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1249.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2308.95</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1615.16</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">807.58</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">50.47</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">40.38</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">302.84</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1249.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">4065.43</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2308.95</Data></Cell>
    <Cell ss:StyleID="s68"/>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21"/>
   <Row ss:AutoFitHeight="0" ss:Height="33.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"><Data ss:Type="String">2022年4月xxx公司托管人员五险一金明细表</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:StyleID="s65"><Data ss:Type="String">部门</Data></Cell>
    <Cell ss:StyleID="s65"><Data ss:Type="String">序号</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果组织名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果职位名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">工伤单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗补充保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">单位社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">备注</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">市场运营部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">6</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">小赵</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">市场运营部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">庆房测绘公司市场运营部部长</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">851.82</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">212.96</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">53.24</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1280</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">2398.02</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1703.65</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">851.82</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">53.24</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">42.59</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">319.43</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1280</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">4250.7299999999996</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">2398.02</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:MergeAcross="4" ss:StyleID="m1392413137156"><Data ss:Type="String">小计：</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">851.82</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">212.96</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">53.24</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1280.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2398.02</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1703.65</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">851.82</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">53.24</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">42.59</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">319.43</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">1280.00</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">4250.73</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">2398.02</Data></Cell>
    <Cell ss:StyleID="s68"/>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21"/>
   <Row ss:AutoFitHeight="0" ss:Height="33.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"><Data ss:Type="String">2022年5月xxx公司托管人员五险一金明细表</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="21">
    <Cell ss:StyleID="s65"><Data ss:Type="String">部门</Data></Cell>
    <Cell ss:StyleID="s65"><Data ss:Type="String">序号</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果组织名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">核算结果职位名称</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">养老保险单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">失业单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">工伤单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">医疗补充保险（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">公积金单位部分（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">单位社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">个人社保合计（社保）</Data></Cell>
    <Cell ss:StyleID="s66"><Data ss:Type="String">备注</Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">7</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">张三</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">测管办测绘员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">284.06</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">71.02</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.75</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">396</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">768.83</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">568.12</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">284.06</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.75</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">14.2</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">106.52</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">396</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1386.65</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">768.83</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">8</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">李四</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">测管办测绘员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">283.17</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">70.790000000000006</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.7</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">347</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">718.66</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">566.33000000000004</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">283.17</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.7</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">14.16</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">106.19</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">347</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1334.55</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">718.66</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">9</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">王五</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">测管办测绘员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">281.52</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">70.38</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.600000000000001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">371</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">740.5</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">563.04</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">281.52</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.600000000000001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">14.08</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">105.57</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">371</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1352.81</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">740.5</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">10</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">小磊</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">测管办测绘员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">286.02999999999997</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">71.510000000000005</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.88</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">371</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">746.42</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">572.04999999999995</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">286.02999999999997</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">17.88</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">14.3</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">107.26</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">371</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1368.52</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">746.42</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:AutoFitHeight="0" ss:Height="28.5">
    <Cell ss:StyleID="s67"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="Number">11</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">小洋</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">托管部</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String">测管办测绘员</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">289.64</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">72.41</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">18.100000000000001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">395</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">775.15</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">579.28</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">289.64</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">18.100000000000001</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">14.48</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">108.62</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">395</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">1405.12</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="Number">775.15</Data></Cell>
    <Cell ss:StyleID="s68"><Data ss:Type="String"></Data></Cell>
   </Row>
   <Row ss:Index="32" ss:AutoFitHeight="0" ss:Height="27.75">
    <Cell ss:MergeAcross="18" ss:StyleID="s64"/>
   </Row>
  </Table>
  <WorksheetOptions xmlns="urn:schemas-microsoft-com:office:excel">
   <PageSetup>
    <Layout x:Orientation="Landscape"/>
    <Header x:Margin="0.51180555555555596"/>
    <Footer x:Margin="0.51180555555555596"/>
    <PageMargins x:Left="1.5743055555555601" x:Right="7.8472222222222193E-2"/>
   </PageSetup>
   <Unsynced/>
   <Print>
    <ValidPrinterInfo/>
    <PaperSizeIndex>9</PaperSizeIndex>
    <Scale>55</Scale>
    <HorizontalResolution>600</HorizontalResolution>
    <VerticalResolution>600</VerticalResolution>
   </Print>
   <Selected/>
   <TopRowVisible>9</TopRowVisible>
   <Panes>
    <Pane>
     <Number>3</Number>
     <ActiveRow>22</ActiveRow>
     <RangeSelection>R23C1:R23C19</RangeSelection>
    </Pane>
   </Panes>
   <ProtectObjects>False</ProtectObjects>
   <ProtectScenarios>False</ProtectScenarios>
  </WorksheetOptions>
  <PageBreaks xmlns="urn:schemas-microsoft-com:office:excel">
   <RowBreaks>
    <RowBreak>
     <Row>6</Row>
    </RowBreak>
    <RowBreak>
     <Row>11</Row>
    </RowBreak>
    <RowBreak>
     <Row>16</Row>
    </RowBreak>
    <RowBreak>
     <Row>21</Row>
    </RowBreak>
   </RowBreaks>
   <x:ColBreaks/>
  </PageBreaks>
 </Worksheet>
</Workbook>

```
## 测试如下
```java
    public static void main(String[] args)  throws Exception {
        XmlConvertExcel xmlConvertExcel=new XmlConvertExcel();
        xmlConvertExcel.xmlToExcel("F:\\excels\\bbb.xml","F:\\excels\\bbb.xlsx");
    }
```

# 四、效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/35f1c62fdff94ae287e28660889b2956.png)

---
参考：
[How to load old Microsoft Office XML file (Excel) using Java](https://stackoverflow.com/questions/7089039/how-to-load-old-microsoft-office-xml-file-excel-using-java)
[Reading Excel Spreadsheet XML](https://github.com/jbaysolutions/excel-xml-reader)