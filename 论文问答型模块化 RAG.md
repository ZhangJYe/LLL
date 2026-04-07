# 论文问答型模块化 RAG

![image-20260328175342989](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260328175342989.png)

你的 RAG 面向什么场景

主要面向论文问答

主要文档类型是什么

文档主要是pdf和md以及text文档

你准备用什么技术栈（比如 Python + LangChain/LlamaIndex + Qdrant 之类）

使用LangChain+milvus+prefect+rag等



它要解决的问题包括：

- 用户上传或导入论文 PDF / md / txt
- 系统解析**论文内容和结构**
- 把论文**分块并建立索引**
- 用户提问后，系统从相关论文中找证据
- 最终生成**带引用、尽量可信**的答案

## 主线 1：离线索引

负责“准备知识”

## 主线 2：在线问答

负责“根据知识回答问题”



```
用户论文(PDF/MD/TXT)
        ↓
[文档接入]
        ↓
[文档解析与标准化]
        ↓
[结构感知切块]
        ↓
[元数据增强]
        ↓
[向量化与索引存储]
        ↓
--------------------------------
        ↓
      用户提问
        ↓
[查询预处理]
        ↓
[查询分类/改写]
        ↓
[混合检索]
        ↓
[重排]
        ↓
[上下文组装]
        ↓
[答案生成]
        ↓
[答案校验/引用格式化]
        ↓
      返回答案
```

## Flow 1：`index_papers_flow`

这是离线索引流。

### 目标

把一批论文处理后写入索引。

### 可以拆成这些 task

1. `load_documents_task`
2. `parse_documents_task`
3. `extract_sections_task`
4. `chunk_documents_task`
5. `enrich_metadata_task`
6. `embed_chunks_task`
7. `write_milvus_task`
8. `write_docstore_task`
9. `log_index_stats_task`

### 每一步为什么拆开

因为你将来会遇到：

- PDF 解析失败
- 某篇论文切块策略要改
- embedding 模型要替换
- Milvus 写入失败要重试

拆开后才能局部重跑。

## Flow 2：`answer_query_flow`

这是在线问答流。

### 目标

回答用户一个问题。

### 可以拆成这些 task

1. `preprocess_query_task`
2. `classify_query_task`
3. `rewrite_query_task`
4. `retrieve_candidates_task`
5. `rerank_candidates_task`
6. `build_context_task`
7. `generate_answer_task`
8. `verify_answer_task`
9. `format_answer_task`
10. `log_query_trace_task`

### 为什么这么拆

因为在线问答调试时，你最想知道的是：

- 是 query rewrite 出问题？
- 还是检索没召回？
- 还是 rerank 排错了？
- 还是 prompt 没写好？

模块化的价值就在这里。