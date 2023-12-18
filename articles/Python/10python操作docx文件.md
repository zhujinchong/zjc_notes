# window下doc docx互转，转pdf

```
from win32com import client


# 转换doc为pdf
def doc2pdf(fn):
    word = client.Dispatch("Word.Application")  # 打开word应用程序
    # for file in files:
    doc = word.Documents.Open(fn)  # 打开word文件
    doc.SaveAs("{}.pdf".format(fn[:-4]), 17)  # 另存为后缀为".pdf"的文件，其中参数17表示为pdf
    doc.Close()  # 关闭原来word文件
    word.Quit()


# 转换docx为pdf
def docx2pdf(fn):
    word = client.Dispatch("Word.Application")  # 打开word应用程序
    # for file in files:
    doc = word.Documents.Open(fn)  # 打开word文件
    doc.SaveAs("{}.pdf".format(fn[:-5]), 17)  # 另存为后缀为".pdf"的文件，其中参数17表示为pdf    
    doc.Close()  # 关闭原来word文件
    word.Quit()


docx2pdf(r'C:\Users\asuka\Desktop\新建文件夹\1.docx')
doc2pdf(r'C:\Users\asuka\Desktop\新建文件夹\2.doc')

```

# linux下doc docx互转，转pdf

安装

```undefined
yum install libreoffice
```

测试

```
libreoffice --version
```

命令

```
libreoffice --headless --convert-to pdf --outdir /your/output/dir /your/doc_docx_wps/files/*.{dosx,doc,wps}
```

单个文件

```
libreoffice --headless --convert-to pdf path-to-your-doc.doc
```

用python脚本执行

```
import os
os.system("libreoffice --headless --convert-to txt path-to-your-doc.doc")
```



# linux下doc docx互站，转pdf

上次发现有一个doc文件libreoffice识别不了，pandoc也识别不了。

最后用一款商业软件aspose解决了。

下载（从PyPI下载whl解决）

```
pip install aspose-words
```

使用：有水印！！

```
import aspose.words as aw
from aspose.words import Document

doc = aw.Document("00.doc")
doc.save("output.docx", aw.SaveFormat.DOCX)
```



# pdf转docx

在linux和window下都支持pdf2docx。

安装

```bash
pip install pdf2docx
pip install pymupdf
pip install python-docx
```

代码

```python
from pdf2docx import Converter

pdf_file = '/path/to/sample.pdf'
docx_file = 'path/to/sample.docx'

# convert pdf to docx
cv = Converter(pdf_file)
cv.convert(docx_file) # 默认参数start=0, end=None
cv.close()
```



# 解析pdf

安装

```
pip install pdfplumber
```





# 解析docx

安装（很好用，推荐）

```
pip install python-docx
```

遍历文本、图片、表格

```
import docx
from docx.document import Document
from docx.oxml.table import CT_Tbl
from docx.oxml.text.paragraph import CT_P
from docx.parts.image import ImagePart
from docx.table import Table, _Cell
from docx.text.paragraph import Paragraph
import io


# 该行只能有一个图片
def is_image(graph: Paragraph, doc: Document):
    images = graph._element.xpath('.//pic:pic')  # 获取所有图片
    print(images)
    for image in images:
        for img_id in image.xpath('.//a:blip/@r:embed'):  # 获取图片id
            part = doc.part.related_parts[img_id]  # 根据图片id获取对应的图片
            if isinstance(part, ImagePart):
                return True
    return False


# 获取图片（该行只能有一个图片）
def get_ImagePart(graph: Paragraph, doc: Document):
    images = graph._element.xpath('.//pic:pic')  # 获取所有图片
    for image in images:
        for img_id in image.xpath('.//a:blip/@r:embed'):  # 获取图片id
            part = doc.part.related_parts[img_id]  # 根据图片id获取对应的图片
            if isinstance(part, ImagePart):
                return part
    return None


def iter_block_items(parent):
    if isinstance(parent, Document):
        parent_elm = parent.element.body
    elif isinstance(parent, _Cell):
        parent_elm = parent._tc
    else:
        raise ValueError("something's not right")

    for child in parent_elm.iterchildren():
        if isinstance(child, CT_P):
            paragraph = Paragraph(child, parent)
            if is_image(paragraph, parent):
                yield get_ImagePart(paragraph, parent)
            else:
                yield Paragraph(child, Paragraph)
        elif isinstance(child, CT_Tbl):
            yield Table(child, Table)


doc = docx.Document("00.docx")
for part in iter_block_items(doc):
    if isinstance(part, Paragraph):
        _ctx = part.text.strip()
        print(_ctx)
    elif isinstance(part, Table):
        table_ctx = []
        for row in part.rows:
            for cell in row.cells:
                _ctx = cell.text.strip()
                table_ctx.append(_ctx)
        print(table_ctx)
    elif isinstance(part, ImagePart):
        io_file = io.BytesIO(part.blob)
        # tmp = ocr_img_io(io_file)
        print('img')

```



