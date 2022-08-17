---
title: TypeError: expected str, bytes or os.PathLike object,not FileStorage 文件上传  Flask报错
date: 2020-07-23 15:12:40
tags: python web
categories: python加油鸭
---

<!--more-->

 -     上传一个文件 file 本来想通过open\(\)来打开文件进行处理的，但是却报错了

```python
@app.route('/getfile', methods=['POST'])
def getfile():
    request_data = request.files['file']
    rsrcmgr = PDFResourceManager()
    retstr = io.StringIO()
    codec = 'utf-8'
    laparams = LAParams()
    device = TextConverter(rsrcmgr, retstr, codec=codec, laparams=laparams)
    fp = open(request_data, 'rb')
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    password = ""
    maxpages = 0
    caching = True
    pagenos = set()

    for page in PDFPage.get_pages(fp, pagenos, maxpages=maxpages,
                                  password=password,
                                  caching=caching,
                                  check_extractable=True):
        interpreter.process_page(page)

    text = retstr.getvalue()

    fp.close()
    device.close()
    retstr.close()
return text
```

 -     报错

```bash
line 27, in getfile fp = open(request_data, 'rb').decode("utf-8") TypeError: expected str, bytes or os.PathLike object, not FileStorage
```

-  解决方案

> The `request.files['file']` is an instance of a [FileStorage](http://werkzeug.pocoo.org/docs/0.14/datastructures/#werkzeug.datastructures.FileStorage) class \(see also <http://flask.pocoo.org/docs/0.12/api/#flask.Request.files>\), so you can't do the `fp = open(request_data, 'rb')`. The FileStorage object contains a `stream` attribute that should point to an open temporary file, and probably you can pass that to `PDFPage.get_pages()`

> 也就是说：Files \[‘ file’\]是 FileStorage 类的一个实例，所以不能执行 fp = open \(request \_ data，‘ rb’\)
> 
> FileStorage 对象包含 `stream`  属性，该属性应该指向一个打开的临时文件，您可以将其传递给 PDFPage.get \_ pages \(\)

 

```python
@app.route('/getfile', methods=['POST'])
def getfile():
    file = request.files['file']
    rsrcmgr = PDFResourceManager()
    retstr = io.StringIO()
    codec = 'utf-8'
    laparams = LAParams()
    device = TextConverter(rsrcmgr, retstr, codec=codec, laparams=laparams)
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    password = ""
    maxpages = 0
    caching = True
    pagenos = set()

    for page in PDFPage.get_pages(file.stream, pagenos, maxpages=maxpages,
                                  password=password,
                                  caching=caching,
                                  check_extractable=True):
        interpreter.process_page(page)

    text = retstr.getvalue()

    device.close()
    retstr.close()
return text
```