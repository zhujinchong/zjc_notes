# Gradio

## 入门

Gradio是一个快速demo的框架，写一个函数即可用。这篇把Gradio基本知识点全覆盖了：

https://blog.csdn.net/sinat_39620217/article/details/130343655



## 最佳性能调优

Gradio官方文档 https://gradio.app/setting-up-a-demo-for-maximum-performance/ 中写了，如何最大化性能。

1、用`queue()`

作用：

* 防止post请求超时，用websockes通信

* 增加交互体验：预测请求的时间

参数：concurrency_count

参数说明：并发线程数，默认是1，注意：线程越多占用内存越多

参数：max_size

参数说明：队列长度，默认None没有限制。当请求队列达到最大数量，用户会收到队列已满的提示消息，增加交互体验。

参数：api_open

参数说明：允许用户直接调用接口，而不通过队列。默认True，部署应该False。当为True时，可以通过api并发测试，此时请求不通过queue，线程最大数量不受concurrency_count限制，而受launch中的max_threads限制。



最佳实践：

```
demo.queue(concurrency_count=2, max_size=100, api_open=False)
demo.launch(share=False, server_name='0.0.0.0', server_port=7888)
```



2、gr.Interface中的参数

参数：batch和max_batch_size

参数说明：因为深度学习模型可以接受batch数据，所以有这个参数。注意：需要修改输入格式。

bach输入举例：

```py
def trim_words(words, lengths):
    trimmed_words = []
    for w, l in zip(words, lengths):
        trimmed_words.append(w[:int(l)])        
    return [trimmed_words]
```



## 如何记录日志

笔记：官方不支持日志，但是可以把控制台输出以文件形式保存起来



github issue：

https://github.com/gradio-app/gradio/issues/2362



代码：把标准输出写入日志

```
import gradio as gr
import sys


class Logger:
    def __init__(self, filename):
        self.terminal = sys.stdout
        self.log = open(filename, "w")

    def write(self, message):
        self.terminal.write(message)
        self.log.write(message)

    def flush(self):
        self.terminal.flush()
        self.log.flush()

    def isatty(self):
        return False


sys.stdout = Logger("output.log")


def test(x):
    a = 1 / int(x)
    print(a)
    return x


def read_logs():
    sys.stdout.flush()
    with open("output.log", "r") as f:
        return f.read()


with gr.Blocks() as demo:
    with gr.Row():
        input = gr.Textbox()
        output = gr.Textbox()
    btn = gr.Button("Run")
    btn.click(test, input, output)

    logs = gr.Textbox()
    demo.load(read_logs, None, logs, every=1)

demo.queue().launch()
```

## 项目启动

日志记录：

```
# 记录到nohup
print(name,file=sys.stderr)
# 记录到指定文件
print(name,file=open("log_input.log","a"))
```

设置显卡：

```
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
```

项目启动：

```
nohup java -jar test.jar 2>&1 &
nohup python web_demo.py > log_err.log 2>&1 &
```

## 缺点

虚增显存，A6000-48G，ChatGLM2-6B-fp16 12G，一周增至21G。两周就要爆显存。







