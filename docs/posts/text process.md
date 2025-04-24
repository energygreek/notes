---
title: text process
comments: true
date: 2025-01-16 17:55:10
tags: []
---

# 文本处理


## sed 将vmstat的输出转成csv

vmstat的输出如下，空格分割的。sed命令`sed 's/^  *//;s/  */,/g'  vmstat.dat > vmstat.02.csv`做了两件事：
1. 将行首的空格去掉
2. 将其他空格转成','符合csv的格式

多个sed命令用';'分隔，且共用/g命令

```
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 1  0 999208 3603968 353020 8258580    0    0    22   166 1016    2  4  1 94  0  0  0
```
处理后
```
procs,-----------memory----------,---swap--,-----io----,-system--,-------cpu-------
r,b,swpd,free,buff,cache,si,so,bi,bo,in,cs,us,sy,id,wa,st,gu
1,0,999208,3603968,353020,8258580,0,0,22,166,1016,2,4,1,94,0,0,0
```

## awk 遍历

我之前的awk通常用来打印某一列的内容，例如`cat vmstat.dat | awk -F' ' '{print $1}'` 打印$1第一列, -F设定分隔符。除此之外还能遍历查找。
例如查找'TARGET'
```sh
awk '{for(i=1;i<=NF;i++)if($i=="TARGET"){print "TARGET located at row: " NR " col: " i }}' 
```

这里NR表示当前的`行`， NF代表当前的`列`。 NR还可以用来指定要处理的行号，例如vmstat中有两个header。例如
1. 只打印第二行的命令是`awk '(NR==2){print $0}'`
2. 要知道'id'在哪一列，则遍历第二行的每一列数据 `awk '(NR==2){for(i=1;i<NF;i++) if($i=="id"){print i}'` 会打印'15'
3. 要打印id这列的第一行的值, `awk '(NR==2){for(i=1;i<NF;i++) if($i=="id"){getline; print $i}'` 会打印'94'

要打印id这列的第N行的值时，就循环调用getline N次。

## 使用python库openpyxl 处理excel

体验了 openpyxl, 接口设计非常自然， 功能也非常全， 包括合并格子、字体、颜色。典型用法如下
```py
import openpyxl


workbook = openpyxl.load_workbook('output.xlsx')
print(workbook.sheetnames)

sheet = workbook['Sheet']
print(sheet.max_row) #总行数
print(sheet.max_column) #总列数
print(sheet.dimensions) #左上角:右下角

cell1 = sheet['A1']
print(cell1.row) #1
print(cell1.column) #A
print(cell1.coordinate) #A1

# 单元格的内容
print(cell1.value)

# 遍历
#先行后列
for row in sheet.iter_rows():
  for c in row:
    print(f"[{c.row}][{c.column}] = {c.value}")

#先列后行
for col in sheet.iter_cols():
  for r in col:
    print(f"[{r.row}][{r.column}] = {r.value}")

# 保存
workbook.save('test.xlsx')
```




