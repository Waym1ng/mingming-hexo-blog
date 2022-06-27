---
title: pandas to_excel() 写入 excel 的几种情况(覆盖,新增,追加,对齐)
date: 2022-06-26 11:25:38
tags:
    - pandas
categories:
    - python
---

## pandas to_excel() 写入 excel 的几种情况(覆盖,新增,追加,对齐)
```
# 以下代码默认已经导入 np,pd
import numpy as np
import pandas as pd
# 执行下面示例之前,最好先删除 ./test.xlsx 文件
```
### 1.覆盖所有原有数据,只保留最后一份数据:
```
# pandas:1.4.1 openpyxl:3.0.9
# 删除文件原有数据,只保留 s2 一份数据(最后一份)
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
s1.to_excel('test.xlsx', sheet_name="111", index=False)
s2.to_excel('test.xlsx', sheet_name="222", index=False) # 只保留此份数据
```
### 2.覆盖所有原有数据,保留当前写入的多份数据:
```
# pandas:1.4.1 openpyxl:3.0.9
# 删除文件原有数据,同时保留s1 和s2 两份数据
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
with pd.ExcelWriter("test.xlsx") as writer:
# 保留两份数据
s1.to_excel(writer, sheet_name="111", index=False)
s2.to_excel(writer, sheet_name="222", index=False)
```
### 3.保留原有数据,新开一个sheet 写入数据
```
# pandas:1.4.1 openpyxl:3.0.9
# 保留原有数据 s1,新开一个sheet 写入数据 s2
from openpyxl import load_workbook
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
# 先写入 s1 的数据(会新建excel文件)
s1.to_excel('test.xlsx', sheet_name='111', index=False)
book = load_workbook("test.xlsx") # 该文件必须存在,并且该语句必须在 with pd.ExcelWriter() 之前
with pd.ExcelWriter("test.xlsx") as writer:
writer.book = book
s2.to_excel(writer, sheet_name="222", index=False)
# 新增一个sheet 并写入,如果这里这里指定的sheet已经存在,那么会在该名称后追加1,2,3,... 创建一个新的sheet写入,不会在原有sheet上修改
```
### 4.重写指定sheet数据,保留原有的其余sheet数据
```
# pandas:1.4.1 openpyxl:3.0.9
# 重写指定sheet数据,保留原有的其余sheet数据
from openpyxl import load_workbook
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
s3 = pd.DataFrame(np.array([['s3', 's3', 's3', 's3']]), columns=['a', 'b', 'c', 'd'])
with pd.ExcelWriter("test.xlsx") as writer:
# 先写入两个sheet
s1.to_excel(writer, sheet_name="111", index=False)
s2.to_excel(writer, sheet_name="222", index=False)
book = load_workbook("test.xlsx")
with pd.ExcelWriter("test.xlsx") as writer:
writer.book = book
writer.sheets = {
i.title: i for i in book.worksheets} # 指定sheet
s3.to_excel(writer, sheet_name="111", index=False)
```
### 5.修改指定sheet内的部分数据,其余保持不变
```
# pandas:1.4.1 openpyxl:3.0.9
# 修改指定sheet内的部分数据,其余保持不变
from openpyxl import load_workbook
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
with pd.ExcelWriter("test.xlsx") as writer:
# 先写入两个sheet
s1.to_excel(writer, sheet_name="111", index=False)
s2.to_excel(writer, sheet_name="222", index=False)
book = load_workbook("test.xlsx")
with pd.ExcelWriter("test.xlsx") as writer:
writer.book = book
sheet = book['222'] # 通过sheet名称 获取 sheet
sheet.cell(2, 1, 'hello') # 修改第二行第一列的值
sheet['b2'] = '你好' # 修改 b2 单元格的值
```

### 6.向 sheet 中追加数据 Excel 中追加
```
# pandas:1.4.1 openpyxl:3.0.9
# 向sheet中追加数据(一),在Excel 中追加
from openpyxl import load_workbook
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
# s4 只有3列,并且列顺序被打乱,以模拟新数据与元数据的差异
s4 = pd.DataFrame(np.array([['s4b', 's4d', 's4c']]), columns=['b', 'd', 'c'])
with pd.ExcelWriter("test.xlsx") as writer:
# 先写入两个sheet
s1.to_excel(writer, sheet_name="111", index=False)
s2.to_excel(writer, sheet_name="222", index=False)
df = pd.read_excel('test.xlsx', sheet_name='111')
row = df.shape[0] # 获取原数据的行数
# 将 新数据 格式化成原数据的模样,以解决数据列之间的差异
s4 = pd.concat([pd.DataFrame(columns=df.columns), s4], ignore_index=True)
book = load_workbook("test.xlsx")
with pd.ExcelWriter("test.xlsx") as writer:
writer.book = book
writer.sheets = {
sheet.title: sheet for sheet in book.worksheets}
# 追加新数据,追加前必须先格式化新数据,否则新数据缺少列,或是列顺序不对会导致数据紊乱
s4.to_excel(writer, sheet_name='111', startrow=row + 1, index=False, header=False)
```

### 7.向sheet中追加数据 pandas中追加数据后,重写指定sheet
```
# pandas:1.4.1 openpyxl:3.0.9
# 向sheet中追加数据(二),在pandas中追加数据后,重写指定sheet
from openpyxl import load_workbook
s1 = pd.DataFrame(np.array([['s1', 's1', 's1', 's1']]), columns=['a', 'b', 'c', 'd'])
s2 = pd.DataFrame(np.array([['s2', 's2', 's2', 's2']]), columns=['a', 'b', 'c', 'd'])
# s4 只有3列,并且列顺序被打乱,以模拟新数据与元数据的差异
s4 = pd.DataFrame(np.array([['s4b', 's4d', 's4c']]), columns=['b', 'd', 'c'])
with pd.ExcelWriter("test.xlsx") as writer:
s1.to_excel(writer, sheet_name="111", index=False)
s2.to_excel(writer, sheet_name="222", index=False)
df = pd.read_excel('test.xlsx', sheet_name='111')
df = pd.concat([df, s4], ignore_index=True) # 合并数据
book = load_workbook("test.xlsx")
with pd.ExcelWriter("test.xlsx") as writer:
writer.book = book
writer.sheets = {
sheet.title: sheet for sheet in book.worksheets}
df.to_excel(writer, sheet_name='111', index=False) # 重写sheet
```