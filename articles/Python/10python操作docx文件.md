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



# pdf转docx

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

安装

```
pip install python-docx
```