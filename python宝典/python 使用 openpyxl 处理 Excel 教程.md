[TOC]

# 前言
python 操作excel 的库有很多 ，有的库只能读取 xsl 格式，比如 xlrd 库；
有的库只能写 xsl 格式，比如 xlwt 库;
有的只能读写 xslx  格式，比如 openpyxl 库 。

综合各库及 xslx  格式比较常见，所以本文主要讲解 openpyxl 库对 xslx  格式的 excel 操作。

# 一、安装openpyxl库
```python
pip install openpyxl
```
# 二、新建excel及写入单元格
## 1.创建一个xlsx格式的excel文件并保存
```python
from openpyxl import Workbook

# 创建一个工作表
wb = Workbook()
# 保存为本地excel文件
wb.save("F:\pythonTest\sample.xlsx")
```
## 2.保存成流(stream)
例如当使用 Pyramid, Flask 或 Django 等 web 应用程序时，如果想把文件保存成流，可以使用 NamedTemporaryFile( )方法
```python
from tempfile import NamedTemporaryFile
from openpyxl import Workbook

wb = Workbook()
with NamedTemporaryFile() as tmp:
     wb.save(tmp.name)
     tmp.seek(0)
     stream = tmp.read()
```

## 3.写入单元格

```python
from openpyxl import Workbook

wb = Workbook()
# 创建第一个sheet
sheet = wb.active

# 方式一 在第1行第1列的单元格写入111
sheet['A1'] = 111
# 方式二 在第1行第1列的单元格写入111
# sheet.cell(row=1, column=1, value=111)
# 方式三 先获取到单元格，再将数据写入value属性
# cell = sheet['A1']
# cell.value = 111

wb.save("F:\pythonTest\sample.xlsx")
```

下面的简单示例将新建一个 xlsx 文件并在里面新建 3 个 sheet 工作表，每个 sheet 工作表都写上一点数据：

```python
from openpyxl import Workbook

# 初始化
wb = Workbook()

# 创建第一个sheet
sheet = wb.active
# 在第1行第1列的单元格写入111
sheet['A1'] = 111

# 创建一个新的sheet
sheet1 = wb.create_sheet()
# 在这个新的sheet中第2行第2列的单元格写入222
sheet1['B2'] = 222

# 创建一个叫"第3个工作表"的新sheet
sheet3 = wb.create_sheet("第3个工作表")
# 在这个新的sheet中第3行第3列的单元格写入333
sheet3['C3'] = 333

# 保存本地excel文件中
wb.save("F:\pythonTest\sample.xlsx")
```
![请添加图片描述](https://img-blog.csdnimg.cn/a82404a7aaf2432db01737c74f497291.jpeg)


# 三、创建sheet工作表及操作

1.创建新的工作表

一个工作表至少有一个工作表，你可以通过 Workbook.active 来创建第一个sheet

```python
sheet = wb.active
```
创建新的带有名字的工作表

```python
# 在结尾插入一个叫 "我的sheet1" 的sheet
sheet1 = wb.create_sheet("我的sheet1") 
# 或者在最开始插入一个叫 "我的sheet12" 的sheet
sheet2 = wb.create_sheet("我的sheet12", 0)
# 或者在倒数第二的位置插入一个叫 "我的sheet13" 的sheet
sheet3 = wb.create_sheet("我的sheet13", -1) 
```
2.删除指定工作表

```python
# 删除名字是 "我的sheet13" 的工作表
wb.remove(wb['3号sheet'])
```

3.修改sheet工作表的名称

```python
# 在结尾插入一个叫 "我的sheet1" 的工作表
sheet1 = wb.create_sheet("我的sheet1") 
# 将"我的sheet1" 的工作表名字改为"first"
sheet1.title = "first"
```

4.获取所有工作表的名称

```python
print(wb.sheetnames) #['Sheet', '我的sheet1', 'first', 'second']
```
5.切换工作表

```python
current = wb["first"]
# 切换到 "second" 工作表
current = wb["second"]
```
6.遍历所有sheet工作表

```python
for s in wb:
     print(s.title)
```
7.复制sheet工作表

```python
sheet = wb.active
sheetNew = wb.copy_worksheet(sheet)
```
# 四、读取excel和单元格
## 1.读取 excel 文件

读取 excel 文件并打印所有的 sheet 工作表名称

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
print(wb.sheetnames) # ['Sheet', 'Sheet1', 'New Title']
```

## 2.读取单元格

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取"New Title" 这个sheet工作表
sheet = wb["第3个工作表"]

# 方式一 获取 "C3"这个单元格
cell = sheet["C3"]
# 方式二 获取 第3行3列 这个单元格
# cell = sheet.cell(row=3, column=3)

# 打印单元格的值
print(cell.value)
```
## 3.获取某一行某一列的数据

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample1.xlsx")
sheet = wb["Sheet"]

# 获取第2行的单元格数据并打印
row = sheet['2']
row_data = []
for cell in row:
    row_data.append(cell.value)
print(row_data)

# 获取第3列的单元格数据并打印
col = sheet['C']
col_data = []
for cell in col:
    col_data.append(cell.value)
print(col_data)


```

## 4.遍历所有单元格

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 遍历所有的sheet工作表
for sheet in wb:
    # 获取当前sheet工作表的行数据
    rows = tuple(sheet.rows)
    # 遍历每一行
    for row in rows:
        # 遍历每一个单元格
        for cell in row:
            print(cell.value)
```
上面使用了sheet.rows 获取行数据，如果要获取列数据可以使用 sheet.columns 

## 5.遍历指定行列范围的单元格

方式一：使用切片方式

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取叫"Sheet"的工作表
sheet = wb["Sheet"]
# 获取叫'A2'到'C4'范围的所有单元格
rows = sheet['A2':'C4']
for cells in rows:
    for cell in cells:
        print(cell.value, end=" ")
    print()
```

方式二：使用iter_rows方法

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取叫"Sheet"的工作表
sheet = wb["Sheet"]
# 获取 2到4行，1到3列 范围的所有单元格
for row in sheet.iter_rows(min_row=2, min_col=1, max_col=3, max_row=4):
    for cell in row:
        print(cell.value, end=" ")
    print()
```

# 五、合并、拆分单元格和插入删除行列

## 1.合并单元格
合并单元格时，除了左上角的单元格内容，其他都选中范围单元格将从工作表中删除
```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取叫"Sheet"的工作表
sheet = wb["Sheet"]

# 方式一 合并 A2到C4 范围的单元格
sheet.merge_cells('A2:C4')
# 方式二 合并 2到4行，1到3列 范围的所有单元格
# sheet.merge_cells(start_row=2, start_column=1, end_row=4, end_column=3)

wb.save("F:\pythonTest\sample.xlsx")
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2312d8badab9452880bc439418d7db50.png)

## 2.拆分合并的单元格
```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取叫"Sheet"的工作表
sheet = wb["Sheet"]

# 方式一 拆分 A2到C4 范围的单元格
sheet.unmerge_cells('A2:C4')
# 方式二 拆分 2到4行，1到3列 范围的所有单元格
ws.unmerge_cells(start_row=2, start_column=1, end_row=4, end_column=4)

wb.save("F:\pythonTest\sample.xlsx")
```

## 3.插入行和列

### (1)插入单行单列

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
sheet = wb["Sheet"]
# 在第3行前插入一行
sheet.insert_rows(3)
# 在第2列前插入一列
sheet.insert_cols(2)
wb.save("F:\pythonTest\sample.xlsx")
```
### (2)插入多行多列
```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
sheet = wb["Sheet"]
# 在第3行前插入四行
sheet.insert_rows(3, 4)
# 在第2列前插入五列
sheet.insert_cols(2, 5)
wb.save("F:\pythonTest\sample.xlsx")
```

## 4.删除行和列
### (1)删除单行单列

```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
sheet = wb["Sheet"]
# 删除第3行
sheet.delete_rows(3)
# 删除第2列
sheet.delete_cols(2)
wb.save("F:\pythonTest\sample.xlsx")
```
### (2)删除多行多列
```python
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
sheet = wb["Sheet"]
# 从第3行前开始删除四行
sheet.delete_rows(3, 4)
# 从第2列开始删除五列
sheet.delete_cols(2, 5)
wb.save("F:\pythonTest\sample.xlsx")
```

# 六、单元格对齐

## 1. 对齐方式与换行

```python
# horizontal是水平方向，vertical 是垂直方向
Alignment(horizontal="center", vertical="center")
# 默认单元格填满了是不换行的，如果要自动换行可使用 wrap_text=True
Alignment(horizontal="center", vertical="center", wrap_text=True)
```

horizontal 水平对齐可选值如下

|   参数值   |   对齐方式   |
| ---- | ---- |
|  left    |   左对齐   |
|  center   |  左右居中    |
|  right   |   右对齐   |
|  fill    |   填满对齐   |
|  distributed    |  分散对齐    |
|  centerContinously    |  连续居中    |
|  justify    |  两端对齐    |
|  general    |   一般对齐   |

 vertical 垂直对齐可选值如下

|   参数值   |   对齐方式   |
| ---- | ---- |
|  top    |   上对齐   |
|  center   |  左右居中    |
|  bottom   |   下对齐   |
|  distributed    |  分散对齐    |
|  justify    |  两端对齐    |

## 2.单元格对齐

```python
from openpyxl import Workbook
from openpyxl.styles import Alignment

wb = Workbook()
sheet = wb.active

# 在第1行第1列的那个单元格写入123546
cell = sheet['A1']
cell.value = 123546
# 将该单元格对齐方式设置为水平和垂直都居中
cell.alignment = Alignment(horizontal="center", vertical="center")

wb.save("F:\pythonTest\sample.xlsx")
```

## 3.合并后的单元格设置对齐方式
可以改变左上单元格的对齐方式、边框等属性来改变整个合并单元格的对齐方式、边框等属性。
```python
from openpyxl.styles import Alignment
from openpyxl import load_workbook

wb = load_workbook("F:\pythonTest\sample.xlsx")
# 获取叫"Sheet"的工作表
sheet = wb["Sheet"]

#  合并 A2到C4 范围的单元格
sheet.merge_cells('A2:C4')
# 改变左上单元格的对齐方式来改变合并单元格的对齐方式
cell = sheet['A2']
cell.alignment = Alignment(horizontal="center", vertical="center")

wb.save("F:\pythonTest\sample.xlsx")
```

# 七、单元格边框设置

## 1. 边框线条粗细、颜色设置
（1）你可以对单元格上下左右的边框进行设置虚线和实线
```python
Border(
    left=Side(style='thick'),
    bottom=Side(style='mediumDashed'),
    right=Side(style='thin'),
    top=Side(style='dashed'))
```
其中`thin`是细实线，`thick`是粗实线，`dashed`是细虚线，`mediumDashed`是粗虚线.
全部边框线条可选的有如下：

```python
'dashDot','dashDotDot', 'dashed','dotted',
'double','hair', 'medium', 'mediumDashDot', 
'mediumDashDotDot','mediumDashed', 'slantDashDot',
'thick', 'thin'
```

（2）你还可以使用 color 参数调整单元格上下左右边框的颜色
```python
Border(
    left=Side(style='thick', color='00000000'),
    bottom=Side(style='mediumDashed', color='00000000'),
    right=Side(style='thin', color='00000000'),
    top=Side(style='dashed', color='00000000'))
```

可取的颜色参考如下：

![请添加图片描述](https://img-blog.csdnimg.cn/98f1254b13394b1682b9fd0ba4b13cc3.jpeg)


## 2.设置单个单元格边框和颜色

```python
from openpyxl import Workbook
from openpyxl.styles import Border, Side

wb = Workbook()
sheet = wb.active

# 在第1行第1列的那个单元格写入123546
cell = sheet['B2']
cell.value = 123546
# 设置该单元格边框和颜色
cell.border = Border(
    left=Side(style='thick', color='00000000'),
    bottom=Side(style='mediumDashed', color='00000000'),
    right=Side(style='thin', color='00000000'),
    top=Side(style='dashed', color='00000000'))

wb.save("F:\pythonTest\sample1.xlsx")

```

## 3.设置多个单元格整体边框和颜色

想要整体添加边框，有两种方式：

（1） 先合并单元格再对左上角的单元格进行操作，但往往这些单元格都有不同的数据，难以进行合并。
（2） 遍历所有的单元格，分别赋予属性。这种方法比较适用于为所有的单元格赋予相同的属性。

# 八、设置单元格背景颜色和文字字体

## 1.设置单元格背景颜色

```python
from openpyxl import Workbook
from openpyxl.styles import PatternFill

wb = Workbook()
sheet = wb.active

# 在第1行第1列的那个单元格写入123546
cell = sheet['B2']
cell.value = 123546
# 使用fgColor属性16进制颜色填充
cell.fill = PatternFill('solid', fgColor="FF00FF")

wb.save("F:\pythonTest\sample1.xlsx")
```

## 2.设置字体、字体的粗细、大小、颜色
```python
from openpyxl import Workbook
from openpyxl.styles import Font

wb = Workbook()
sheet = wb.active

# 在第1行第1列的那个单元格写入123546
cell = sheet['B2']
cell.value = 123546
# 设置字体样式：  字体大小为30， bold加粗,字体颜色 00FFFF 16进制颜色
cell.font = Font(u'微软雅黑', size=30, bold=True, color="00FFFF")

wb.save("F:\pythonTest\sample1.xlsx")
```

# 九、设置行高和列宽
```python
from openpyxl import Workbook

wb = Workbook()
sheet = wb.active

# 设置第1行高度为60
sheet.row_dimensions[1].height = 60
# 设置B列宽度为30
sheet.column_dimensions["B"].width = 30

wb.save("F:\pythonTest\sample1.xlsx")
```

---
参考：
[Tutorial](https://openpyxl.readthedocs.io/en/stable/tutorial.html)