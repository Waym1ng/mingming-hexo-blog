---
title: 用pandas生成excel文件示例，并调整excel的格式或样式
date: 2022-06-27 09:15:30
tags:
    - pandas
categories:
    - python
---
## 用pandas生成excel
- 当我们有特殊的需求时，比如要修改excel的行宽列宽，还有字体样式等等
### 需求示例
![](/images/ex.png)
### 代码实现
```python
# -*- coding: utf-8 -*-
import pandas as pd
from  datetime import datetime, timedelta


def modify_excel_format(excel_data, writer, df):
    # ----------调整excel格式 ---------------
    workbook = writer.book
    fmt = workbook.add_format({"font_name": u"宋体"})
    col_fmt = workbook.add_format(
        {'bold': True, 'font_size': 11, 'font_name': u'宋体', 'border': 1, 'bg_color': '#0265CB','font_color': 'white',
         'valign': 'vcenter', 'align': 'center'})
    detail_fmt = workbook.add_format(
        {"font_name": u"宋体", 'border': 0, 'valign': 'vcenter', 'align': 'center','font_size': 11, 'text_wrap': True})
    worksheet1 = writer.sheets['Sheet1']
    for col_num, value in enumerate(df.columns.values):
        worksheet1.write(0, col_num, value, col_fmt)
    # 设置列宽行宽
    worksheet1.set_column('A:F', 20, fmt)
    worksheet1.set_row(0, 30, fmt)
    for i in range(1, len(excel_data)+1):
        worksheet1.set_row(i, 27, detail_fmt)


excel_data = []
columns = ["用户", "名字", "标题", "类", "测试", "语言"]
tmp = ["张同学", "张同学", "pd 生成excel", "pandas", "pd", "python"]
excel_data.append(tmp)
df = pd.DataFrame(data=excel_data, columns=columns)
t = datetime.now().date() - timedelta(days=1)
with pd.ExcelWriter(path='demo-%d%02d%02d.xlsx' % (t.year, t.month, t.day), engine="xlsxwriter") as writer:
    df.to_excel(writer, sheet_name='Sheet1', encoding='utf8', header=False, index=False, startcol=0, startrow=1)
    modify_excel_format(excel_data, writer, df)
    writer.save()
    print('ok!')

```