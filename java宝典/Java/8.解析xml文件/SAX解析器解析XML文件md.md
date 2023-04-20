[TOC]

# 一、概述

SAX，也称为**Simple API for XML**，是jdk自带的 用于解析 XML 文档 API。它是一种基于流的解析方式，边读取XML边解析，并以事件回调的方式让调用者获取数据。因为是一边读一边解析，所以无论XML有多大，占用的内存都很小，所以 **SAX 具有高效**的内存管理。

# 二、待解析的xml文件

```xml
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet" xmlns:o="urn:schemas-microsoft-com:office:office">
    <Styles>
        <Style ss:ID="Default" ss:Name="Normal">
            <Alignment ss:Vertical="Bottom"/>
            <Borders/>
            <Font ss:FontName="宋体" x:CharSet="134" ss:Size="12"/>
            <Interior/>
            <NumberFormat/>
            <Protection/>
        </Style>
        <Style ss:ID="s79" ss:Name="常规 2">
            <Alignment ss:Vertical="Center"/>
            <Borders/>
            <Font ss:FontName="等线" x:CharSet="134" ss:Size="11" ss:Color="#000000"/>
            <Interior/>
            <NumberFormat/>
            <Protection/>
        </Style>
    </Styles>
    <Worksheet ss:Name="薪酬发放表1">
        <Row ss:AutoFitHeight="0" ss:Height="33.75">
            <Cell ss:MergeAcross="7" ss:StyleID="s64">
                <Data ss:Type="String">2022年5月某公司托管人员五险一金明细表</Data>
            </Cell>
        </Row>
        <Row ss:AutoFitHeight="0" ss:Height="21">
            <Cell ss:StyleID="s65">
                <Data ss:Type="String">部门</Data>
            </Cell>
            <Cell ss:StyleID="s65" ss:MergeAcross="3" ss:MergeDown="2">
                <Data ss:Type="String">序号</Data>
            </Cell>
            <Cell ss:MergeAcross="1" ss:StyleID="m3181415230988">
                <Data ss:Type="String">姓名</Data>
            </Cell>
            <Cell ss:StyleID="s66" ss:MergeDown="2">
                <Data ss:Type="String">公积金单位部分（社保）</Data>
            </Cell>
            <Cell ss:Index="16371" ss:StyleID="Default"/>
            <Cell ss:StyleID="Default"/>
            <Cell ss:StyleID="Default"/>
        </Row>
        <Row ss:AutoFitHeight="0" ss:Height="21"/>
    </Worksheet>
    <Worksheet ss:Name="薪酬发放表2">
        <Row ss:Height="22.5">
            <Cell ss:MergeAcross="7" ss:StyleID="s64">
                <Data ss:Type="String">2022年6月某公司托管人员五险一金明细表</Data>
            </Cell>
        </Row>
        <Row ss:Index="3">
            <Cell ss:StyleID="s67" ss:MergeAcross="3">
                <Data ss:Type="String">信息中心</Data>
            </Cell>
            <Cell ss:StyleID="s68" ss:Formula=" ">
                <Data ss:Type="Number">1</Data>
            </Cell>
            <Cell ss:StyleID="s68">
                <Data ss:Type="String">张三</Data>
            </Cell>
            <Cell ss:StyleID="s69" ss:MergeDown="6">
                <Data ss:Type="Number">1252.43</Data>
            </Cell>
            <Cell ss:StyleID="s69">
                <Data ss:Type="Number">313.11</Data>
            </Cell>
            <Cell ss:StyleID="s68">
                <Data ss:Type="String"/>
            </Cell>
        </Row>
    </Worksheet>
</Workbook>
```


# 三、XmlRow类

```java
package com.xmlutil;

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

# 四、SAXHandler类

```java
package com.xmlutil;

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

# 五、测试

```java
import com.alibaba.fastjson.JSON;
import com.xmlutil.SAXHandler;
import org.apache.commons.io.IOUtils;
import org.apache.poi.EncryptedDocumentException;
import org.apache.poi.poifs.filesystem.FileMagic;

import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.*;
import java.nio.charset.Charset;

public class MainServer {
    public static void main(String[] args) throws Exception {
        File file = new File("F:\\excels\\textXml.xml");
        SAXParserFactory parserFactor = SAXParserFactory.newInstance();
        SAXParser parser = parserFactor.newSAXParser();
        SAXHandler handler = new SAXHandler();
        String fileContent = IOUtils.toString(new FileInputStream(file), Charset.defaultCharset());
        ByteArrayInputStream bis = new ByteArrayInputStream(fileContent.getBytes());
        parser.parse(bis, handler);

        System.out.println(JSON.toJSONString(handler.xmlSheetList));
    }
```

输出：

```java
[
	[{
		"cellList": ["2022年5月某公司托管人员五险一金明细表"],
		"cellMergeAcrossList": [7],
		"cellMergeDownList": [0]
	}, {
		"cellList": ["部门", "序号", "姓名", "公积金单位部分（社保）"],
		"cellMergeAcrossList": [0, 3, 1, 0, 0, 0, 0],
		"cellMergeDownList": [0, 2, 0, 2, 0, 0, 0]
	}, {
		"cellList": [],
		"cellMergeAcrossList": [],
		"cellMergeDownList": []
	}],
	[{
		"cellList": ["2022年6月某公司托管人员五险一金明细表"],
		"cellMergeAcrossList": [7],
		"cellMergeDownList": [0]
	}, {
		"cellList": ["信息中心", "1", "张三", "1252.43", "313.11", ""],
		"cellMergeAcrossList": [3, 0, 0, 0, 0, 0],
		"cellMergeDownList": [0, 0, 0, 6, 0, 0]
	}]
]
```

---

参考：

[Parsing an XML File Using SAX Parser](https://www.baeldung.com/java-sax-parser)

[Parsing an XML File Using StAX](https://www.baeldung.com/java-stax)

[excel-xml-reader](https://github.com/jbaysolutions/excel-xml-reader)

[使用SAX](https://www.liaoxuefeng.com/wiki/1252599548343744/1320418577219618)