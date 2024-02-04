参考：

[babyagi: 人工智能任务管理系统 - 掘金 (juejin.cn)](https://juejin.cn/post/7218815501433946173)

[babyagi/README.md at main · yoheinakajima/babyagi (github.com)](https://github.com/yoheinakajima/babyagi/blob/main/README.md)



背景：

BabyAGI 是 Twitter 上分享的 [Task-Driven Autonomous Agent](https://twitter.com/yoheinakajima/status/1640934493489070080?s=20)（2023 年 3 月 28 日）的精简版本。这个版本减少到 140 行：13 个注释、22 个空白和 105 个代码。为什么叫BabyAGI，因为原作者不认为只是AGI。

作者发推的时间：2023.3.29



原理：

GPT分解任务，并存储在Pinecone的队列中。按队列解决任务。直至任务不能再被分解。