问：如何记录日志？

答：用Python中的logging模块：https://blog.csdn.net/Runner1st/article/details/96481954

问：如何解决跨域？

答：用FaskApi中的中间件(CORSMiddleware)：https://juejin.cn/post/7035788888450793485

问：如何流式返回结果？

答：用*WwebSocket*或*Server-sent Events* ，这里是*Server-sent Events*的实现: https://blog.csdn.net/kidom1412/article/details/130886719

问：是否要用异步处理？

答：https://blog.csdn.net/yyw794/article/details/108859240

总结：

1、一步一步教你用*Server-sent Events* 实现ChatGLM2-6b接口：https://blog.csdn.net/kidom1412/article/details/130886719

2、用*Server-sent Events* 实现ChatGLM2-6b接口代码全：

# FaskApi

uvicorn api:app --host '192.168.1.131' --port 7888

uvicorn api:app --host '192.168.1.131' --port 7888 --workers 2



这个包含多种微调方式，我用的其中的Lora方法：https://github.com/hiyouga/ChatGLM-Efficient-Tuning/blob/main/README_zh.md

这个只有QLoRA微调代码，也可以改成LoRA训练（我是直接用QLoRA训练，导致模型量化了，最后效果不好。需要改代码。）：https://github.com/shuxueslpi/chatGLM-6B-QLoRA/tree/main/data