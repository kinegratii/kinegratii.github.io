---
title: docx和docxtpl库的基本使用
date: 2020-03-22 08:42:43
categories: 技术研究
tags:
- 文档
---

在web开发过程中，经常需要导出一些word文档，比如运营月报、进货单、收据等。它们的特点是格式都是一致的，只是需要不同的数据进行填充。

docx 和 docxtpl 是比较常见的处理 word 文件的python库。其中前者用于创建一个包含段落、图片、表格、页眉等元素文档，后者使用类似 jinja2 的方式从模板文档生成新的文档。

> docx是一种压缩文档格式，使用XML形式定义了样式、段落等内容。详情可以查看 [《An Informal Introduction to DOCX》](https://www.toptal.com/xml/an-informal-introduction-to-docx)。

<!-- more -->

# 文档构建

## 安装方式

使用 pip 安装这两个库即可：

```shell
pip install python-docx
pip install docxtpl
```

其中 docxtpl 还依赖 jinja2 库。

## 创建 word

使用以下的方式创建一个 Document 对象，之后的操作都是基于该对象。

```python
from docx import Document

document = Document()
```

使用 `add_*` 函数添加到该文档对象，包括：

| 函数                  | 说明   |
| --------------------- | ------ |
| add_paragraph         | 段落   |
| add_heading           | 标题   |
| add_page_break        | 换页符 |
| add_table(rows, cols) | 表格   |
| add_picture           | 图片   |
| add_run               | 字符片段     |

run 是word中包含相同样式的字符片段，通常它的级别比 paragraph 更低。



这是正常的文字。**这是粗体文字。** 这是正常的文字。



 比如上述的一段文字。依次由3个 run 组成。



待文档构建完成后，可以使用 save 函数保存到文件中。

```python
document.save('demo.docx')
```



# 文档模板

## 使用方法

docxtpl 的使用方法和 jinja2 类似。

```python
from docxtpl import DocxTemplate

doc = DocxTemplate("my_word_template.docx")
context = { 'company_name' : "World company" }
doc.render(context)
doc.save("generated_doc.docx")
```



docxtpl 的核心在于前端标签定义。

## 变量渲染

就像 jinja2 中一样，使用两个大括号即可。

```
{{ <var> }}
```

当 var 是 RichText 对象时，必须使用以下的方式：

```
{{r <var> }}
```

为了能够准确的区分以上两种情况，建议变量名称不要单独命名为 r ，。


# 存储和输出

## 整合Flask

将文档对象保存到新建的 `BytesIO()` 实例即可，但需要注意的是在返回给浏览器前将文件指针重新指向开头。

详情查看 flask.send_file 的用法。

```python
import io

from docxtpl import DocxTemplate
from flask import send_file

def export_monthly_report(year, index):
    context = {} # 填充数据
    doc = DocxTemplate(app.root_path + '/templates/report_monthly.docx')
    target_filename = 'monthly_{}_{}.docx'.format(year, index)
    doc.render(context)
    file_stream = io.BytesIO()
    doc.save(file_stream)
    file_stream.seek(0)
    return send_file(file_stream, as_attachment=True, attachment_filename=target_filename)
```

### 整合Django

和 flask类似。

```python
def export_monthly_report(request, year, index):
    context = {} # 填充数据
    doc = DocxTemplate(app.root_path + '/templates/report_monthly.docx')
    target_filename = 'monthly_{}_{}.docx'.format(year, index)
    doc.render(context)
    file_stream = io.BytesIO()
    doc.save(file_stream)
    file_stream.seek(0)
    response = HttpResponse(
        file_stream.getvalue(),  # use the stream's contents
        content_type="application/vnd.openxmlformats-officedocument.wordprocessingml.document",
    )
    response["Content-Disposition"] = 'attachment; filename = "{}"'.format(target_filename)
    response["Content-Encoding"] = "UTF-8"
    return response
```
