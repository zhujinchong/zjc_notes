# FastAPI请求

## web服务代码

```
import time
import shutil
import os
from fastapi import FastAPI, File, UploadFile
from fastapi.responses import StreamingResponse, FileResponse
import io
import base64
import uvicorn
import json

app = FastAPI()


def image_to_base64():
    with open('test.png', "rb") as image_file:
        # 读取图片文件的二进制数据
        image_binary = image_file.read()
        # 进行Base64编码（byte）
        encoded_image = base64.b64encode(image_binary)
        # 将字节数据转换为字符串
        encoded_image_string = encoded_image.decode('utf-8')
    return encoded_image_string


# 流式返回数据
@app.get("/stream")
def stream_combined():
    def data_generator():
        for i in range(10000):
            time.sleep(0.05)
            # yield str(i) + "\n"
            yield json.dumps({
                "text": str(i),
                "img": image_to_base64()
            }, ensure_ascii=False) + "\n"

    # 这两种返回数据的格式都可以（一种是二进制，一种是文本，还可以指定其它的如音视频等）
    # return StreamingResponse(data_generator(), media_type="application/octet-stream")
    return StreamingResponse(data_generator(), media_type="text/plain")


# 上传文件的例子
@app.post("/uploadfile/")
def create_upload_file(file: UploadFile = File(...)):
    # 指定保存文件的路径
    save_path = "uploads"
    os.makedirs(save_path, exist_ok=True)

    # 将上传的文件保存到指定路径
    file_path = os.path.join(save_path, file.filename)
    with open(file_path, "wb") as file_object:
        shutil.copyfileobj(file.file, file_object)
    return {"filename": file.filename}


# 返回文件的例子
@app.post("/downloadfile/")
def read_item(arg_dict: dict):
    filename = arg_dict.get("filename")
    return FileResponse(filename)


uvicorn.run(app=app, host='0.0.0.0', port=8000, workers=1)
```



## request请求测试

```
import requests
import json


class RequestHandler:
    def post(self, url, **kwargs):
        #  URL 后面的键值对，形式为 ?key1=value1&key2=value2
        # 此时，faskapi方法参数中必须定义好key
        params = kwargs.get("params")
        # 发送表单数据
        # 此时，faskapi方法参数中必须定义好key
        data = kwargs.get("data")
        # 发送的 JSON 数据。Content-Type 标头将被设置为 application/json。
        # 此时，faskapi方法参数中必须定义好字典dict接收
        json = kwargs.get("json")
        try:
            response = requests.post(url, params=params, data=data, json=json)
            if response.status_code == 200:
                return response.text
            else:
                return ""
        except Exception as e:
            print("post请求错误: %s" % e)

    def stream(self, url, **kwargs):
        params = kwargs.get("params")
        data = kwargs.get("data")
        json = kwargs.get("json")
        try:
            response = requests.post(url, params=params, data=data, json=json)
            for line in response.iter_lines():
                # 如果是二进制，需要decode；如果是文本/json不需要decode
                print(line.decode("utf-8"))
        except Exception as e:
            print("post请求错误: %s" % e)


ret = RequestHandler().post("http://127.0.0.1:8000/downloadfile", params={"filename": "haha"})
print(ret)
```



# 流式传输

## 底层原理

所有请求都是HTTP协议，HTTP协议支持以下几种技术

```
1、分块传输（Chunked Transfer Encoding）：
允许服务器在生成数据的同时将其发送给客户端，而不需要等待整个响应完成。这使得客户端可以在接收到一部分数据时就开始处理，而不必等待整个响应完成。
分块传输通常通过设置响应头的 Transfer-Encoding: chunked 来实现。

2、WebSocket协议
WebSockets 是一种在单个 TCP 连接上进行全双工通信的协议，允许服务器和客户端之间进行实时双向数据传输。通过建立持久连接，服务器可以异步地向客户端推送数据，而不需要等待客户端的请求。
WebSockets 在实现实时通信、推送通知等场景时非常有用。

3、Server-Sent Events (SSE)
SSE 是一种用于服务器向客户端推送实时事件的技术，基于普通的 HTTP 或 HTTPS 连接。服务器可以异步地向客户端发送事件，而客户端通过 EventSource API 接收这些事件。
SSE 主要用于单向通信，适用于实时通知和事件推送。
```

## FastAPI的StreaingResponse是哪种技术

FastAPI的StreamingResponse用的基于 Starlette 框架提供的StreamingResponse。

StreamingResponse是Starlette 框架中用于支持流式传输的响应类型。它的实现基于 ASGI（Asynchronous Server Gateway Interface）。

增强知识点：

```
ASGI（Asynchronous Server Gateway Interface）是一个异步的Python Web服务器接口标准。类似于Java中的Java Servlet 规范。
FastAPI类似于Java中的SpringBoot开发框架。
uvicorn支持ASGI的服务器，所以可以运行FastAPI。就像Tomcat是可以运行Java Web的容器。
```

问题1：服务器和HTTP协议什么关系

答：所有请求最开始都是HTTP协议，这些请求被转发到Web服务器。Web服务器增强了静态网页成动态网页。

问题2：StreamingResponse到底用的是HTTP哪项技术？

答：在 FastAPI 中，当你使用 StreamingResponse 返回数据时，它会将响应头中的 `Transfer-Encoding` 设置为 `chunked`，并且通过异步生成器函数逐块产生数据。所以，FastAPI是通过分块技术实现的。

问题3：ASGI的异步有什么用？

答： ASGI 允许应用程序和服务器以异步的方式进行通信。这使得能够有效地处理大量并发连接，而不会阻塞整个应用程序。对于需要处理数千个并发连接的应用，异步的处理方式比传统的同步方式更为高效。等等，还有其他优点。

问题4：前端怎么接收

Axios接受

