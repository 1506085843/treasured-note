[TOC]

# 一、 实现方式一
## 1、所需的依赖

maven文件如下：

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

<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.18</version>
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
    <version>1.21</version>
</dependency>
```



## 2、工具类

```java


import org.apache.commons.io.FileUtils;
import org.apache.poi.hssf.usermodel.*;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFCellStyle;
import org.apache.poi.xssf.usermodel.XSSFColor;
import org.apache.poi.xssf.usermodel.XSSFFont;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.text.DecimalFormat;
import java.text.SimpleDateFormat;
import java.util.*;

public class Excel2HtmlUtil {
    /**
     *
     * @param filePath    excel源文件文件的路径
     * @param htmlPositon 生成的html文件的路径
     * @param isWithStyle 是否需要表格样式 包含 字体 颜色 边框 对齐方式
     * @throws Exception
     *
     */
    public static String readExcelToHtml(String filePath, String htmlPositon, boolean isWithStyle,String type,String attname) throws Exception {
        InputStream is = null;
        String htmlExcel = null;
        Map<String,String> stylemap = new HashMap<String,String>();
        try {
            if("csv".equalsIgnoreCase(type)) {
                htmlExcel = getCSVInfo(filePath,htmlPositon);
                writeFile1(htmlExcel, htmlPositon,stylemap,attname);
            }else {
                File sourcefile = new File(filePath);
                is = new FileInputStream(sourcefile);
                Workbook wb = WorkbookFactory.create(is);
                if (wb instanceof XSSFWorkbook) { // 03版excel处理方法
                    XSSFWorkbook xWb = (XSSFWorkbook) wb;
                    htmlExcel = getExcelInfo(xWb, isWithStyle,stylemap);
                } else if (wb instanceof HSSFWorkbook) { // 07及10版以后的excel处理方法
                    HSSFWorkbook hWb = (HSSFWorkbook) wb;
                    htmlExcel = getExcelInfo(hWb, isWithStyle,stylemap);
                }
                writeFile(htmlExcel, htmlPositon,stylemap,attname);
            }
        } catch (Exception e) {
            System.out.println("文件被损坏或不能打开，无法预览");
            //throw new Exception("文件被损坏或不能打开，无法预览");
        } finally {
            try {
                if(is!=null)
                    is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return htmlPositon;
    }

    private static void getcscvvalue(BufferedReader reader,List col,String oldvalue,List list) {
        String line = null;
        try {
            while((line=reader.readLine())!=null){
                String[] item = line.split(",",-1);
                boolean isbreak = false;
                for(int i=0;i<item.length;i++) {
                    String value = item[i];
                    if(value.endsWith("\"")) {
                        value = oldvalue+value;
                        col.add(value);
                    }else if(item.length==1) {
                        value = oldvalue+value;
                        getcscvvalue(reader,col,value,list);
                        isbreak = true;
                    }else if(value.startsWith("\"")){
                        getcscvvalue(reader,col,value,list);
                        isbreak = true;
                    }else {
                        col.add(value);
                    }
                }
                if(!isbreak) {
                    list.add(col);
                    col = new ArrayList();
                }
            }
        } catch (IOException e) {
        }
    }

    private static String getCSVInfo(String filePath,String htmlPositon) {
        StringBuffer sb = new StringBuffer();
        DataInputStream in = null;
        try {
            in=new DataInputStream(new FileInputStream(filePath));
            BufferedReader reader=new BufferedReader(new InputStreamReader(in));
            //reader.readLine();
            String line = null;
            List list = new ArrayList();
            while((line=reader.readLine())!=null){
                String[] item = line.split(",");
                List col = new ArrayList();
                for(int i=0;i<item.length;i++) {
                    String value = item[i];
                    if(value.startsWith("\"")) {
                        getcscvvalue(reader,col,value,list);
                    }else {
                        col.add(value);
                    }
                }
                list.add(col);
            }
            sb.append("<table>");
            for(int i=0;i<list.size();i++) {
                List col = (List) list.get(i);
                if(col==null||col.size()==0) {
                    sb.append("<tr><td ></td></tr>");
                }
                sb.append("<tr>");
                for(int j=0;j<col.size();j++) {
                    String value = (String) col.get(j);
                    if (value == null||"".equals(value)) {
                        sb.append("<td> </td>");
                        continue;
                    }else {
                        sb.append("<td>"+value+"</td>");
                    }
                }
                sb.append("</tr>");
            }
            sb.append("</table>");
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return sb.toString();
    }

    //读取excel文件，返回转换后的html字符串
    private static String getExcelInfo(Workbook wb, boolean isWithStyle, Map<String,String> stylemap) {
        StringBuffer sb = new StringBuffer();
        StringBuffer ulsb = new StringBuffer();
        ulsb.append("<ul>");
        int num = wb.getNumberOfSheets();
        //遍历excel文件里的每一个sheet
        for(int i=0;i<num;i++) {
            Sheet sheet = wb.getSheetAt(i);// 获取第i个Sheet的内容
            String sheetName = sheet.getSheetName();
            if(i==0) {
                ulsb.append("<li id='li_"+i+"' class='cur' onclick='changetab("+i+")'>"+sheetName+"</li>");
            }else {
                ulsb.append("<li id='li_"+i+"' onclick='changetab("+i+")'>"+sheetName+"</li>");
            }
            int lastRowNum = sheet.getLastRowNum();
            Map<String, String> map[] = getRowSpanColSpanMap(sheet);
            Map<String, String> map1[] = getRowSpanColSpanMap(sheet);
            sb.append("<table id='table_"+i+"' ");
            if(i==0) {
                sb.append("class='block'");
            }
            sb.append(">");
            Row row = null; // 兼容
            Cell cell = null; // 兼容

            int maxRowNum = 0;
            int maxColNum = 0;
            //遍历每一行
            for (int rowNum = sheet.getFirstRowNum(); rowNum <= lastRowNum; rowNum++) {
                row = sheet.getRow(rowNum);
                if (row == null) {
                    continue;
                }
                int lastColNum = row.getLastCellNum();
                for (int colNum = 0; colNum < lastColNum; colNum++) {
                    cell = row.getCell(colNum);
                    if (cell == null) { // 特殊情况 空白的单元格会返回null
                        continue;
                    }
                    String stringValue = getCellValue1(cell);
                    if (map1[0].containsKey(rowNum + "," + colNum)) {
                        map1[0].remove(rowNum + "," + colNum);
                        if(maxRowNum<rowNum) {
                            maxRowNum = rowNum;
                        }
                        if(maxColNum<colNum) {
                            maxColNum = colNum;
                        }
                    } else if (map1[1].containsKey(rowNum + "," + colNum)) {
                        map1[1].remove(rowNum + "," + colNum);
                        if(maxRowNum<rowNum) {
                            maxRowNum = rowNum;
                        }
                        if(maxColNum<colNum) {
                            maxColNum = colNum;
                        }
                        continue;
                    }
                    if (stringValue == null || "".equals(stringValue.trim())) {
                        continue;
                    }else {
                        if(maxRowNum<rowNum) {
                            maxRowNum = rowNum;
                        }
                        if(maxColNum<colNum) {
                            maxColNum = colNum;
                        }
                    }
                }
            }
            for (int rowNum = sheet.getFirstRowNum(); rowNum <= maxRowNum; rowNum++) {
                row = sheet.getRow(rowNum);
                if (row == null) {
                    sb.append("<tr><td ></td></tr>");
                    continue;
                }
                sb.append("<tr>");
                int lastColNum = row.getLastCellNum();
                for (int colNum = 0; colNum <= maxColNum; colNum++) {
                    cell = row.getCell(colNum);
                    if (cell == null) { // 特殊情况 空白的单元格会返回null
                        sb.append("<td> </td>");
                        continue;
                    }
                    String stringValue = getCellValue(cell);
                    if (map[0].containsKey(rowNum + "," + colNum)) {
                        String pointString = map[0].get(rowNum + "," + colNum);
                        map[0].remove(rowNum + "," + colNum);
                        int bottomeRow = Integer.valueOf(pointString.split(",")[0]);
                        int bottomeCol = Integer.valueOf(pointString.split(",")[1]);
                        int rowSpan = bottomeRow - rowNum + 1;
                        int colSpan = bottomeCol - colNum + 1;
                        sb.append("<td rowspan= '" + rowSpan + "' colspan= '" + colSpan + "' ");
                    } else if (map[1].containsKey(rowNum + "," + colNum)) {
                        map[1].remove(rowNum + "," + colNum);
                        continue;
                    } else {
                        sb.append("<td ");
                    }

                    // 判断是否需要样式
                    if (isWithStyle) {
                        dealExcelStyle(wb, sheet, cell, sb,stylemap);// 处理单元格样式
                    }

                    sb.append("><nobr>");
                    //如果单元格为空要判断该单元格是不是通过其他单元格计算得到的
                    if (stringValue == null || "".equals(stringValue.trim())) {
                        FormulaEvaluator evaluator = wb.getCreationHelper().createFormulaEvaluator();
                        if (evaluator.evaluate(cell) != null) {
                            //如果单元格的值是通过其他单元格计算来的，则通过单元格计算获取
                            String cellnumber = evaluator.evaluate(cell).getNumberValue() + "";
                            //如果单元格的值是小数，保留两位
                            if (null != cellnumber && cellnumber.contains(".")) {
                                String[] decimal = cellnumber.split("\\.");
                                if (decimal[1].length() > 2) {
                                    int num1 = decimal[1].charAt(0) - '0';
                                    int num2 = decimal[1].charAt(1) - '0';
                                    int num3 = decimal[1].charAt(2) - '0';
                                    if (num3 == 9) {
                                        num2 = 0;
                                    } else if (num3 >= 5) {
                                        num2 = num2 + 1;
                                    }
                                    cellnumber = decimal[0] + "." + num1 + num2;
                                }
                            }
                            stringValue = cellnumber;
                        }
                        sb.append(stringValue.replace(String.valueOf((char) 160), " "));
                    } else {
                        // 将ascii码为160的空格转换为html下的空格（ ）
                        sb.append(stringValue.replace(String.valueOf((char) 160), " "));
                    }
                    sb.append("</nobr></td>");
                }
                sb.append("</tr>");
            }
            sb.append("</table>");
        }
        ulsb.append("</ul>");
        return ulsb.toString()+sb.toString();
    }


    private static Map<String, String>[] getRowSpanColSpanMap(Sheet sheet) {
        Map<String, String> map0 = new HashMap<String, String>();
        Map<String, String> map1 = new HashMap<String, String>();
        int mergedNum = sheet.getNumMergedRegions();
        CellRangeAddress range = null;
        for (int i = 0; i < mergedNum; i++) {
            range = sheet.getMergedRegion(i);
            int topRow = range.getFirstRow();
            int topCol = range.getFirstColumn();
            int bottomRow = range.getLastRow();
            int bottomCol = range.getLastColumn();
            map0.put(topRow + "," + topCol, bottomRow + "," + bottomCol);
            // System.out.println(topRow + "," + topCol + "," + bottomRow + "," +
            // bottomCol);
            int tempRow = topRow;
            while (tempRow <= bottomRow) {
                int tempCol = topCol;
                while (tempCol <= bottomCol) {
                    map1.put(tempRow + "," + tempCol, "");
                    tempCol++;
                }
                tempRow++;
            }
            map1.remove(topRow + "," + topCol);
        }
        Map[] map = { map0, map1 };
        return map;
    }

    private static String getCellValue1(Cell cell) {
        String result = new String();
        switch (cell.getCellType()) {
            case NUMERIC:// 数字类型
                result = "1";
                break;
            case STRING:// String类型
                result = "1";
                break;
            case BLANK:
                result = "";
                break;
            default:
                result = "";
                break;
        }
        return result;
    }

    /**
     * 获取表格单元格Cell内容
     *
     * @param cell
     * @return
     */
    private static String getCellValue(Cell cell) {
        String result = new String();
        switch (cell.getCellType()) {
            case NUMERIC:// 数字类型
                if (DateUtil.isCellDateFormatted(cell)) {// 处理日期格式、时间格式
                    SimpleDateFormat sdf = null;
                    if (cell.getCellStyle().getDataFormat() == HSSFDataFormat.getBuiltinFormat("h:mm")) {
                        sdf = new SimpleDateFormat("HH:mm");
                    } else {// 日期
                        sdf = new SimpleDateFormat("yyyy-MM-dd");
                    }
                    Date date = cell.getDateCellValue();
                    result = sdf.format(date);
                } else if (cell.getCellStyle().getDataFormat() == 58) {
                    // 处理自定义日期格式：m月d日(通过判断单元格的格式id解决，id的值是58)
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
                    double value = cell.getNumericCellValue();
                    Date date = org.apache.poi.ss.usermodel.DateUtil.getJavaDate(value);
                    result = sdf.format(date);
                } else {
                    double value = cell.getNumericCellValue();
                    CellStyle style = cell.getCellStyle();
                    DecimalFormat format = new DecimalFormat();
                    String temp = style.getDataFormatString();
                    // 单元格设置成常规
                    if (temp.equals("General")) {
                        format.applyPattern("#");
                    }
                    result = format.format(value);
                }
                break;
            case STRING:// String类型
                result = cell.getRichStringCellValue().toString();
                break;
            case BLANK:
                result = "";
                break;
            default:
                result = "";
                break;
        }
        return result;
    }

    /**
     * 处理表格样式
     *
     * @param wb
     * @param sheet
     * @param sb
     */
    private static void dealExcelStyle(Workbook wb, Sheet sheet, Cell cell, StringBuffer sb,Map<String,String> stylemap) {
        CellStyle cellStyle = cell.getCellStyle();
        if (cellStyle != null) {
            HorizontalAlignment alignment = cellStyle.getAlignment();
            // sb.append("align='" + convertAlignToHtml(alignment) + "' ");//单元格内容的水平对齐方式
            VerticalAlignment verticalAlignment = cellStyle.getVerticalAlignment();
            String _style = "vertical-align:"+convertVerticalAlignToHtml(verticalAlignment)+";";
            if (wb instanceof XSSFWorkbook) {
                XSSFFont xf = ((XSSFCellStyle) cellStyle).getFont();
                //short boldWeight = xf.getBoldweight();
                short boldWeight = 400;
                String align = convertAlignToHtml(alignment);
                int columnWidth = sheet.getColumnWidth(cell.getColumnIndex());
                _style +="font-weight:" + boldWeight + ";font-size: " + xf.getFontHeight() / 2 + "%;width:" + columnWidth + "px;text-align:" + align + ";";

                XSSFColor xc = xf.getXSSFColor();
                if (xc != null && !"".equals(xc)) {
                    _style +="color:#" + xc.getARGBHex().substring(2) + ";";
                }

                XSSFColor bgColor = (XSSFColor) cellStyle.getFillForegroundColorColor();
                if (bgColor != null && !"".equals(bgColor)) {
                    _style +="background-color:#" + bgColor.getARGBHex().substring(2) + ";"; // 背景颜色
                }
                _style +=getBorderStyle(0, cellStyle.getBorderTop().getCode(),((XSSFCellStyle) cellStyle).getTopBorderXSSFColor());
                _style +=getBorderStyle(1, cellStyle.getBorderRight().getCode(),((XSSFCellStyle) cellStyle).getRightBorderXSSFColor());
                _style +=getBorderStyle(2, cellStyle.getBorderBottom().getCode(),((XSSFCellStyle) cellStyle).getBottomBorderXSSFColor());
                _style +=getBorderStyle(3, cellStyle.getBorderLeft().getCode(),((XSSFCellStyle) cellStyle).getLeftBorderXSSFColor());
            } else if (wb instanceof HSSFWorkbook) {
                HSSFFont hf = ((HSSFCellStyle) cellStyle).getFont(wb);
                short boldWeight = hf.getFontHeight();
                short fontColor = hf.getColor();
                HSSFPalette palette = ((HSSFWorkbook) wb).getCustomPalette(); // 类HSSFPalette用于求的颜色的国际标准形式
                HSSFColor hc = palette.getColor(fontColor);
                String align = convertAlignToHtml(alignment);
                int columnWidth = sheet.getColumnWidth(cell.getColumnIndex());
                _style +="font-weight:" + boldWeight + ";font-size: " + hf.getFontHeight() / 2 + "%;text-align:" + align + ";width:" + columnWidth + "px;";
                String fontColorStr = convertToStardColor(hc);
                if (fontColorStr != null && !"".equals(fontColorStr.trim())) {
                    _style +="color:" + fontColorStr + ";"; // 字体颜色
                }
                short bgColor = cellStyle.getFillForegroundColor();
                hc = palette.getColor(bgColor);
                String bgColorStr = convertToStardColor(hc);
                if (bgColorStr != null && !"".equals(bgColorStr.trim())) {
                    _style +="background-color:" + bgColorStr + ";"; // 背景颜色
                }
                _style +=getBorderStyle(palette, 0, cellStyle.getBorderTop().getCode(), cellStyle.getTopBorderColor());
                _style +=getBorderStyle(palette, 1, cellStyle.getBorderRight().getCode(), cellStyle.getRightBorderColor());
                _style +=getBorderStyle(palette, 3, cellStyle.getBorderLeft().getCode(), cellStyle.getLeftBorderColor());
                _style +=getBorderStyle(palette, 2, cellStyle.getBorderBottom().getCode(), cellStyle.getBottomBorderColor());
            }
            String calssname="";
            if(!stylemap.containsKey(_style)) {
                int count = stylemap.size();
                calssname = "td"+count;
                stylemap.put(_style, calssname);
            }else {
                calssname = stylemap.get(_style);
            }
            if(!"".equals(calssname)) {
                sb.append("class='"+calssname+"'");
            }
        }
    }

    /**
     * 单元格内容的水平对齐方式
     *
     * @param alignment
     * @return
     */
    private static String convertAlignToHtml(HorizontalAlignment alignment) {
        String align = "center";
        switch (alignment) {
            case LEFT:
                align = "left";
                break;
            case CENTER:
                align = "center";
                break;
            case RIGHT:
                align = "right";
                break;
            default:
                break;
        }
        return align;
    }

    /**
     * 单元格中内容的垂直排列方式
     *
     * @param verticalAlignment
     * @return
     */
    private static String convertVerticalAlignToHtml(VerticalAlignment verticalAlignment) {

        String valign = "middle";
        switch (verticalAlignment) {
            case BOTTOM:
                valign = "bottom";
                break;
            case CENTER:
                valign = "middle";
                break;
            case TOP:
                valign = "top";
                break;
            default:
                break;
        }
        return valign;
    }

    private static String convertToStardColor(HSSFColor hc) {
        StringBuffer sb = new StringBuffer("");
        if (hc != null) {
            if (HSSFColor.HSSFColorPredefined.AUTOMATIC.getIndex() == hc.getIndex()) {
                return null;
            }
            sb.append("#");
            for (int i = 0; i < hc.getTriplet().length; i++) {
                sb.append(fillWithZero(Integer.toHexString(hc.getTriplet()[i])));
            }
        }
        return sb.toString();
    }

    private static String fillWithZero(String str) {
        if (str != null && str.length() < 2) {
            return "0" + str;
        }
        return str;
    }

    static String[] bordesr = { "border-top:", "border-right:", "border-bottom:", "border-left:" };
    static String[] borderStyles = { "solid ", "solid ", "solid ", "solid ", "solid ", "solid ", "solid ", "solid ",
            "solid ", "solid", "solid", "solid", "solid", "solid" };

    private static String getBorderStyle(HSSFPalette palette, int b, short s, short t) {
        if (s == 0)
            return bordesr[b] + borderStyles[s] + "#d0d7e5 1px;";
        String borderColorStr = convertToStardColor(palette.getColor(t));
        borderColorStr = borderColorStr == null || borderColorStr.length() < 1 ? "#000000" : borderColorStr;
        return bordesr[b] + borderStyles[s] + borderColorStr + " 1px;";
    }

    private static String getBorderStyle(int b, short s, XSSFColor xc) {
        if (s == 0)
            return bordesr[b] + borderStyles[s] + "#d0d7e5 1px;";
        if (xc != null && !"".equals(xc)) {
            String borderColorStr = xc.getARGBHex();// t.getARGBHex();
            borderColorStr = borderColorStr == null || borderColorStr.length() < 1 ? "#000000"
                    : borderColorStr.substring(2);
            return bordesr[b] + borderStyles[s] + borderColorStr + " 1px;";
        }
        return "";
    }

    /*
     * @param content 生成的excel表格标签
     *
     * @param htmlPath 生成的html文件地址
     */
    private static void writeFile(String content, String htmlPath, Map<String,String> stylemap,String name) {
        File file2 = new File(htmlPath);
        StringBuilder sb = new StringBuilder();
        try {
            file2.createNewFile();// 创建文件
            sb.append("<html><head><meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\"><title>"+name+"</title><style type=\"text/css\">");
            sb.append("ul{list-style: none;max-width: calc(100%);padding: 0px;margin: 0px;overflow-x: scroll;white-space: nowrap;} ul li{padding: 3px 5px;display: inline-block;border-right: 1px solid #768893;} ul li.cur{color: #F59C25;} table{border-collapse:collapse;display:none;width:100%;} table.block{display: block;}");
            for(Map.Entry<String, String> entry : stylemap.entrySet()){
                String mapKey = entry.getKey();
                String mapValue = entry.getValue();
                sb.append(" ."+mapValue+"{"+mapKey+"}");
            }
            sb.append("</style><script>");
            sb.append("function changetab(i){var block = document.getElementsByClassName(\"block\");block[0].className = block[0].className.replace(\"block\",\"\");var cur = document.getElementsByClassName(\"cur\");cur[0].className = cur[0].className.replace(\"cur\",\"\");var curli = document.getElementById(\"li_\"+i);curli.className += ' cur';var curtable = document.getElementById(\"table_\"+i);curtable.className=' block';}");
            sb.append("</script></head><body>");
            sb.append("<div>");
            sb.append(content);
            sb.append("</div>");
            sb.append("</body></html>");
            FileUtils.write(file2, sb.toString(),"UTF-8");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void writeFile1(String content, String htmlPath, Map<String,String> stylemap,String name) {
        File file2 = new File(htmlPath);
        StringBuilder sb = new StringBuilder();
        try {
            file2.createNewFile();// 创建文件
            sb.append("<html><head><meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\"><title>"+name+"</title><style type=\"text/css\">");
            sb.append("ul{list-style: none;max-width: calc(100%);padding: 0px;margin: 0px;overflow-x: scroll;white-space: nowrap;} ul li{padding: 3px 5px;display: inline-block;border-right: 1px solid #768893;} ul li.cur{color: #F59C25;} table{border-collapse:collapse;width:100%;} td{border: solid #000000 1px; min-width: 200px;}");
            sb.append("</style></head><body>");
            sb.append("<div>");
            sb.append(content);
            sb.append("</div>");
            sb.append("</body></html>");
            FileUtils.write(file2, sb.toString(),"UTF-8");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}


```

## 3、测试与效果

```java
    public static void main(String[] args) {
        String sourcepath = "D:\\myExcel.xlsx";
        String htmlPositon =  "D:\\测试.html";
        Excel2HtmlUtil.readExcelToHtml(sourcepath, htmlPositon, true, "xlsx", "测试");
    }
```

效果如下：

![请添加图片描述](https://img-blog.csdnimg.cn/8237c1cf57064b79ab3a9680f6d066b9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6KW_5YeJ55qE5oKy5Lyk,size_20,color_FFFFFF,t_70,g_se,x_16)
# 二、实现方式二
通过Aspose.Cells 来实现 excel 转化为 html
[Aspose.Cells官网](https://repository.aspose.com/cells/)

## 1.所需依赖
maven如下：

```xml
    <repositories>
        <repository>
            <id>AsposeJavaAPI</id>
            <name>Aspose Java API</name>
            <url>https://repository.aspose.com/repo/</url>
        </repository>
    </repositories>


        <dependency>
            <groupId>com.aspose</groupId>
            <artifactId>aspose-cells</artifactId>
            <version>22.4</version>
        </dependency>
```
或者[百度云下载jar](https://pan.baidu.com/s/1lElXrUZ4seowJ1xSfAvPLA)
链接：https://pan.baidu.com/s/1lElXrUZ4seowJ1xSfAvPLA 
提取码：wssh

## 2.去除水印工具类
因为评估版生成的html会有水印（[官网水印说明](https://docs.aspose.com/cells/java/licensing/)），所以我写了个工具类来去除水印，工具类如下：

```java

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class ClearUtils {
    //获取文件夹下所有的html或htm文件路径
    public static List<String> getAllFilesPath(String path, List<String> list) {
        File file = new File(path);
        File[] tempList = file.listFiles();
        for (int i = 0; i < tempList.length; i++) {
            String filePath = tempList[i].toString();
            if (tempList[i].isFile()) {
                if (filePath.endsWith(".html") || filePath.endsWith(".htm")) {
                    list.add(filePath);
                }
            } else {
                //如果是文件夹则递归
                getAllFilesPath(filePath, list);
            }
        }
        return list;
    }

    //清除所有sheet中的水印,返回第一个sheet的名称
    public static String clearWatermark(String htmlPath) throws IOException {
        String allhtmlDirPath = htmlPath.replace(".html", "_files");
        //获取所有的htm或html文件路径
        List<String> allFilePath = new ArrayList<>();
        allFilePath = ClearUtils.getAllFilesPath(allhtmlDirPath, allFilePath);
        //对每一个文件进行清除Evaluation Only.
        String firstSheet = "";
        for (String filePath : allFilePath) {
            //按行读取文件内容到lines
            FileInputStream file = new FileInputStream(filePath);
            InputStreamReader in = new InputStreamReader(file, "UTF-8");
            BufferedReader buf = new BufferedReader(in); //加入到缓存区
            String str = "";
            List<String> lines = new ArrayList<>();
            while ((str = buf.readLine()) != null) {
                //该行若含有Evaluation Only.则清除该行
                if (str.contains("Evaluation Only.")) {
                    lines.add("");
                } else {
                    lines.add(str);
                }
            }
            buf.close();
            file.close();
            //去除底部Evaluation Warning按钮
            if (filePath.endsWith("tabstrip.htm")) {
                firstSheet = clearEvaluation(lines);
                //替换为第一个sheet的名称，使得打开html时默认打开第一个sheet
                modefy(htmlPath, firstSheet);
            }
            //将修改的文件内容lines再写回文件
            FileWriter f = new FileWriter(filePath, false); //文件读取为字符流，追加模式写数据
            BufferedWriter bufw = new BufferedWriter(f); //文件加入缓冲区
            for (String li : lines) {
                bufw.write(li); //向缓冲区写入数据
                bufw.write("\n"); //向缓冲区写入换行，或者采用buf.newLine();
            }
            bufw.close(); //关闭缓冲区并将信息写入文件
            f.close();
        }
        return firstSheet;
    }

    //清除多出的Evaluation Warning按钮
    public static String clearEvaluation(List<String> lines) {
        //查找Evaluation Warning出现在lines中的位置
        int eWPosition = -1;
        int lastTdPosition = -1;
        int i = 0;
        int firstSheetPosition = -1;
        for (String s : lines) {
            if (s.contains("Evaluation Warning")) {
                eWPosition = i;
            }
            if (s.contains("<td")) {
                lastTdPosition = i;
            }
            if (s.contains("<a href=\"") && firstSheetPosition == -1) {
                firstSheetPosition = i;
            }
            i++;
        }
        //修改Evaluation Warning样式将其隐藏
        if (eWPosition == lastTdPosition && eWPosition != -1) {
            String table = lines.get(eWPosition);
            int postion = table.lastIndexOf("<td");
            StringBuilder strBuild = new StringBuilder(table);
            strBuild.insert(postion + 4, " style=\"display: none;\"");
            table = String.valueOf(strBuild);
            lines.set(eWPosition, table);
        }
        //获取第一个sheet的名称
        String firstSheet = "";
        if (firstSheetPosition != -1) {
            String table = lines.get(firstSheetPosition);
            int start = table.indexOf("<a href=\"") + 9;
            int end = table.indexOf("\"", start);
            firstSheet = table.substring(start, end);
        }
        return firstSheet;
    }

    public static void modefy(String htmlPath, String firstSheetName) throws IOException {
        //按行读取文件内容到lines
        FileInputStream file = new FileInputStream(htmlPath);
        InputStreamReader in = new InputStreamReader(file, "UTF-8");
        BufferedReader buf = new BufferedReader(in); //加入到缓存区
        String str = "";
        List<String> lines = new ArrayList<>();
        boolean first = false;
        while ((str = buf.readLine()) != null) {
            //该行若含有Evaluation Only.则清除该行
            if (str.contains("<frame src=\"") && !first) {
                String[] sp = str.split("/");
                StringBuffer buffer = new StringBuffer(sp[1]);
                buffer.replace(0, sp[1].indexOf("\""), firstSheetName);
                String reStr = buffer.toString();
                lines.add(sp[0] + "/" + reStr);
                first = true;
            } else {
                lines.add(str);
            }
        }
        buf.close();
        file.close();

        //将修改的文件内容lines再写回文件
        FileWriter f = new FileWriter(htmlPath, false); //文件读取为字符流，追加模式写数据
        BufferedWriter bufw = new BufferedWriter(f); //文件加入缓冲区
        for (String li : lines) {
            bufw.write(li); //向缓冲区写入数据
            bufw.write("\n"); //向缓冲区写入换行，或者采用buf.newLine();
        }
        bufw.close(); //关闭缓冲区并将信息写入文件
        f.close();
    }
}

```
## 3.转化为html

```java
public class MainServer {
    public static void main(String[] args) throws Exception {
		HtmlSaveOptions save = new HtmlSaveOptions(SaveFormat.HTML);
        //加载excel文件
        Workbook book = new Workbook("F:\\excels\\myExcel.xlsx");
        //转化为html
        book.save("F:\\asposeHtml\\myExcelbb.html", save);
        //去除水印
        ClearUtils.clearWatermark("F:\\asposeHtml\\myExcelbb.html");
    }
}
```
## 4.转换效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/b226a8a6bd0a493bb0c3da5e65fec4f6.png)


# 三、 实现方式三
通过Spire.XLS 来实现 excel 转化为 html ,Spire.XLS是一个专业的 Java Excel API，使开发人员无需使用 Microsoft Office 即可创建、管理、操作、转换和打印 Excel 工作表。

（经过测试，当一个excel文件中含有多个sheet时，转化速度会变慢，所以比较推荐实现方式一来转换）
## 1、所需的依赖
通过下面任意一种方式下载spire.xls.jar

- [官网下载jar](https://www.e-iceblue.com/Download/xls-for-java.html)

- 百度云下载 链接：https://pan.baidu.com/s/1wfna9JysI68ll_kbfUgRkA 提取码：bzhs

- 通过Maven下载，Maven的 pom.xml 中配置的存储库和依赖下载如下：
  ([关于免费版的说明](https://www.e-iceblue.com/Introduce/free-xls-for-java.html#.Ynx7FOgzaUk))

```xml
<repositories>
        <repository>
            <id>com.e-iceblue</id>
            <name>e-iceblue</name>
            <url>https://repo.e-iceblue.com/nexus/content/groups/public/</url>
        </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId> e-iceblue </groupId>
        <artifactId>spire.xls.free</artifactId>
        <version>5.1.0</version>
    </dependency>
</dependencies>

```
## 2、转换为html代码如下

```java
import com.spire.xls.FileFormat;
import com.spire.xls.Workbook;
import com.spire.xls.Worksheet;
import com.spire.xls.core.spreadsheet.HTMLOptions;

public class MainServer {
    public static void main(String[] args) {
        //加载excel文件
        Workbook wb = new Workbook();
        wb.loadFromFile("F:\\myExcel.xlsx");

        //将excel 转化为 html (如果excel里有多个sheet,会将多个sheet都转化)
        wb.saveToFile("F:\\ToHtml.html", FileFormat.HTML);

        //将excel 中的某一个 sheet 转化为 html ,这里get(0)即取第一个sheet转化为html
        Worksheet sheet = wb.getWorksheets().get(0);
        HTMLOptions options = new HTMLOptions();
        options.setImageEmbedded(true);
        sheet.saveToHtml("F:\\ToHtmlWithImageEmbeded.html", options);
    }
 }
```
## 3、效果
excel如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0cf0131f2db4aabaaf0641ed33b8c4a.png)
转化为html如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/370f7a751598444a8a869d45f080092d.png)
只转化第一个 sheet 为 html 效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6d3c29edd084b3ea0a0d67cf632e65b.png)

---
其他相关文章：
[Convert Excel Spreadsheets to HTML using the Apache POI library](https://stackoverflow.com/questions/9740215/convert-excel-spreadsheets-to-html-using-the-apache-poi-library)
[java 解析word、excel转换为html、pdf](https://www.cnblogs.com/vofill/p/14841803.html)