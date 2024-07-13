阿里：[Vector | Graph：蚂蚁首个开源Graph RAG框架设计解读 - Florian - 博客园 (cnblogs.com)](https://www.cnblogs.com/fanzhidongyzby/p/18252630/graphrag)



# 1. 概述

RAG的目标是通过知识库增强内容生成的质量，通常做法是将检索出来的文档作为提示词的上下文，一并提供给大模型让其生成更可靠的答案。更进一步地，RAG的整体链路还可以与提示词工程（Prompt Engineering）、模型微调（Fine Tuning）、知识图谱（Knowledge Graph）等技术结合，构成更广义的RAG问答链路。



除了传统意义上的增强内容生成，RAG的理念还可以进一步泛化到链路的其他阶段：

- **增强训练**：[REALM](https://arxiv.org/abs/2002.08909)引入了知识检索器增强大模型预训练，以改进大模型的问答质量和可解释性。
- **增强微调**：[RA-DIT](https://arxiv.org/abs/2310.01352)实现了对大模型和检索器的双指令微调，[RAFT](https://arxiv.org/abs/2403.05313)通过微调让大模型可以识别干扰文档。
- **增强语料**：[MuRAG](https://arxiv.org/abs/2210.02928)支持了多模态数据的检索，提升了大模型在文本/图像混合检索场景下的推理质量。
- **增强知识**：[GraphRAG](https://arxiv.org/abs/2404.16130)使用图社区摘要解决总结性查询任务的问题，将知识图谱技术应用到RAG。
- **增强检索**：[CRAG](https://arxiv.org/abs/2401.15884)通过对检索到的文档置信度进行评估，提升问答上下文的质量。
- **增强推理**：[RAT](https://arxiv.org/abs/2403.05313)在推理阶段将RAG与CoT相结合，以改进长期推理和生成任务的效果。

我们希望向大家分享一下：引入知识图谱技术后，传统RAG链路到Graph RAG链路会有什么样的变化，如何兼容RAG中的向量数据库（Vector Database）和图数据库（Graph Database）基座，以及蚂蚁的Graph RAG开源技术方案和未来优化方向。



# 2. 传统RAG

传统RAG的核心链路分为三个阶段：

- **索引（向量嵌入）**：通过Embedding模型服务实现文档的向量编码，写入向量数据库。
- **检索（相似查询）**：通过Embedding模型服务实现查询的向量编码，使用相似性查询（ANN）实现topK结果搜索。
- **生成（文档上下文）**：Retriver检索的结果文档作为上下文和问题一起提交给大模型处理。

传统RAG希望通过知识库的关联知识增强大模型问答的上下文以提升生成内容质量，但也存在诸多问题。

论文[23]《Seven Failure Points...》总结了传统RAG的7个问题：

1. **知识库内容缺失**：现有的文档其实回答不了用户的问题，系统有时被误导，给出的回应其实是“胡说八道”，理想情况系统应该回应类似“抱歉，我不知道”。
2. **TopK截断有用文档**：和用户查询相关的文档因为相似度不足被TopK截断，本质上是相似度不能精确度量文档相关性。
3. **上下文整合丢失**：从数据库中检索到包含答案的文档，因为重排序/过滤规则等策略，导致有用的文档没有被整合到上下文中。
4. **有用信息未识别**：受到LLM能力限制，有价值的文档内容没有被正确识别，这通常发生在上下文中存在过多的噪音或矛盾信息时。
5. **提示词格式问题**：提示词给定的指令格式出现问题，导致大模型/微调模型不能识别用户的真正意图。
6. **准确性不足**：LLM没能充分利用或者过度利用了上下文的信息，比如给学生找老师首要考虑的是教育资源的信息，而不是具体确定是哪个老师。另外，当用户的提问过于笼统时，也会出现准确性不足的问题。
7. **答案不完整**：仅基于上下文提供的内容生成答案，会导致回答的内容不够完整。比如问“文档 A、B和C的主流观点是什么？”，更好的方法是分别提问并总结。



总的来看：

- 问题1-3：属于知识库工程层面的问题，可以通过完善知识库、增强知识确定性、优化上下文整合策略解决。
- 问题4-6：属于大模型自身能力的问题，依赖大模型的训练和迭代。
- 问题7：属于RAG架构问题，更有前景的思路是使用Agent引入规划能力。



# 3. Graph RAG

Graph RAG的核心链路分如下三个阶段：

- **索引（三元组抽取）**：通过LLM服务实现文档的三元组提取，写入图数据库。比如由蚂蚁和浙大联合研发的大模型知识抽取框架[OneKE](https://github.com/zjunlp/DeepKE/blob/main/example/llm/OneKE.md)在零样本泛化性能上全面超过了现有模型。
- **检索（子图召回）**：通过LLM服务实现查询的关键词提取和泛化（大小写、别称、同义词等），并基于关键词实现子图遍历（DFS/BFS），搜索N跳以内的局部子图。
- **生成（子图上下文）**：将局部子图数据格式化为文本，作为上下文和问题一起提交给大模型处理。



#  4. 通用RAG设计

传统RAG和Graph RAG，可以抽象出一个更通用的RAG结构。

## 4.1 架构设计

图不好看，略。

## 4.2 领域建模

设计不全，略。可看原文参考。

## 4.3 技术选型

综上所述，要构建一个完整的开源Graph RAG链路，离不开三个重要的子系统：一个可以支持RAG的AI工程框架，一个知识图谱系统和一个图存储系统。

* 开源的AI工程框架有诸多选型：[LangChain](https://github.com/langchain-ai/langchain)、[LlamaIndex](https://github.com/run-llama/llama_index)、[RAGFlow](https://github.com/infiniflow/ragflow)、[DB-GPT](https://github.com/eosphoros-ai/DB-GPT)等。
* 知识图谱系统有：[Jena](https://github.com/apache/jena)、[RDF4J](https://github.com/eclipse-rdf4j/rdf4j)、[Oxigraph](https://github.com/oxigraph/oxigraph)、[OpenSPG](https://github.com/OpenSPG/openspg)等。
* 图存储系统有[Neo4j](https://github.com/neo4j/neo4j)、[JanusGraph](https://github.com/JanusGraph/janusgraph)、[NebulaGraph](https://github.com/vesoft-inc/nebula)、[TuGraph](https://github.com/TuGraph-family/tugraph-db)等。

而作为蚂蚁首个对外开源的Graph RAG框架，我们采用**蚂蚁全自主的开源产品**：DB-GPT + OpenSPG + TuGraph。



# 5. 开源技术方案

在DB-GPT的[v0.5.6](https://github.com/eosphoros-ai/DB-GPT/releases/tag/v0.5.6)版本中，我们提供了完整的Graph RAG框架实现（[PR 1506](https://github.com/eosphoros-ai/DB-GPT/pull/1506)）。接下来我们结合这个PR，阐述Graph RAG的关键实现细节。

略。可看原文和DB-GPT代码。



# 6. 优化方向

其实大家在对DB-GPT上Graph RAG实现进行初步的测试后，会发现当下仍有不少体验问题。不避讳的讲，这里除了功能完善度的原因之外，还有Graph RAG自身设计上的不足，这也为后续的进一步优化方向提供了思路。

**文章[26]《From RAG to GraphRAG...》**总结了Graph RAG的不足：

## 6.1 内容索引阶段

Graph RAG的内容索引阶段主要目标便是构建高质量的知识图谱，值得继续探索的有以下方向：

- **图谱元数据**：从文本到知识图谱，是从非结构化信息到结构化信息的转换的过程，虽然图一直被当做半结构化数据，但有结构的LPG（Labeled Property Graph）除了有利于图存储系统的性能优化，还可以协助大模型更好地理解知识图谱的语义，帮助其生成更准确的查询。
- **知识抽取微调**：通用大模型在三元组的识别上实际测试下来仍达不到理想预期，针对知识抽取的微调模型反而表现出更好地效果，如前面提到的OneKE。
- **图社区总结**：这部分源自于微软的Graph RAG的研究工作，通过构建知识图谱时生成图社区摘要，以解决知识图谱在面向总结性查询时“束手无策”的问题。另外，同时结合图社区总结与子图明细可以生成更高质量的上下文。
- **多模态知识图谱**：多模态知识图谱可以大幅扩展Graph RAG知识库的内容丰富度，对客观世界的数据更加友好，浙大的[MyGO](https://arxiv.org/abs/2404.09468)框架提出的方法提升MMKGC（Multi-modal Knowledge Graph Completion）的准确性和可靠性。Graph RAG可以借助于MMKG（Multi-modal Knowledge Graph）和MLLM（Multi-modal Large Language Model）实现更全面的多模态RAG能力。
- **混合存储**：同时使用向量/图等多种存储系统，结合传统RAG和Graph各自的优点，组成混合RAG。参考**文章[27]《GraphRAG: Design Patterns...》**提出的多种Graph RAG架构，如图学习语义聚类、图谱向量双上下文增强、向量增强图谱搜索、混合检索、图谱增强向量搜索等，可以充分利用不同存储的优势提升检索质量。



## 6.2 检索生成阶段

Graph RAG的检索生成阶段主要目标便是从知识图谱上召回高质量上下文，值得继续探索的有以下方向：

- **图语言微调**：使用自然语言在知识图谱上做召回，除了基本的关键词搜索方式，还可以尝试使用图查询语言微调模型，直接将自然语言翻译为图查询语句，这里需要结合图谱的元数据以获得更准确的翻译结果。过去，我们在[Text2GQL](https://mp.weixin.qq.com/s/rZdj8TEoHZg_f4C-V4lq2A)上做了一些初步的工作。
- **混合RAG**：这部分与前边讲过的混合存储是一体的，借助于底层的向量/图/全文索引，结合关键词/自然语言/图语言多种检索形式，针对不同的业务场景，探索高质量Graph RAG上下文的构建。
- **测试验证**：Graph RAG的测试和验证可以参考传统RAG的Benchmark方案，如[RAGAS](https://arxiv.org/abs/2309.15217)、[ARES](https://arxiv.org/abs/2311.09476)、[RECALL](https://arxiv.org/abs/2311.08147)、[RGB](https://arxiv.org/abs/2309.01431)、[CRUD-RAG](https://arxiv.org/abs/2401.17043v2)等。
- **RAG智能体**：从某种意义上说，RAG其实是Agent的简化形式（知识库可以看到Agent的检索工具），同时当下我们也看到RAG对记忆和规划能力的集成诉求（如RAT/RoG等），因此未来RAG向带有记忆和规划能力的智能体架构演进几乎是必然趋势。另外，Agent自身需要的长期记忆存储也会反向依赖RAG的知识库，所以RAG与Agent其实是相辅相成、互相促进的。

 

# 7. 尾记

通过以上介绍，相信大家对RAG到Graph RAG的技术演进有了更进一步的了解，并且基于RAG的索引、检索、生成三个基本阶段抽象出了通用的RAG框架，兼容了Vector、Graph、FullText等多种索引形式，最终在开源技术中完整落地。最后通过探讨Graph RAG未来的优化与演进方向，总结了内容索引和检索生成阶段的不同改进思路，以及RAG向Agent架构的演化趋势。Graph RAG是个相对新颖AI工程领域，需要探索和改进的工作还有很多要做，我们诚邀DB-GPT/OpenSPG/TuGraph的广大开发者们一起参与共建。

前不久Jerry Liu（LlamaIndex CEO）在技术报告《Beyond RAG: Building Advanced Context-Augmented LLM Applications》中也抛出了“RAG的未来是Agent”相似观点。所以，无论是“RAG for Agents”还是“Agents for RAG”，亦或是“从RAG到Graph RAG再到Agents”，目光可及的是智能体将是未来AI应用的主旋律。