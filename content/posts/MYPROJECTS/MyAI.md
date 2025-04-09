---
title: 'MyAIv0.2 - 个性化模型构建工具'
date: 2024-06-22T22:41:20+08:00
draft: false
tags: ['AI','foryyz']
---
> 项目启动时间：2024/06/02
>
> 最后更新时间：2024/11/07
>
> 当前版本：V0.200

# My AI v0.2

(本版本代码暂不开源，MyAI v0.1可前往：[foryyz/Hello-My-AI: YourPersonalAIAssistant](https://github.com/foryyz/Hello-My-AI))

项目名称：**My AI v0.2 - 基于RAG的多模态Agent构建工具**

**迭代日期：2024/06/30 → 2024/11/07 → **

**v0.2 主要特性**：随着GPT4ALL和多模态模型Llama3.2-Vision的发布，支持了工具的**多模态**发布

演示模型：llava-llama3:8B
演示环境：**Ubuntu22.04 - wsl2**



## 0 How To Run

`python -m streamlit run Run.py`



## 1 核心更新内容

![Update.png](../assets/MyAI/Update.png)

### 1.1 检索模型的检索模式

- 将原数据直接存入向量数据库
- **NOW: 将所有原数据建立为摘要模型，存入向量数据库，并使摘要对应原始数据**
- → 利用不同向量空间，存放不同格式数据

### 1.2 [多模态检索器](#extract-data-type)

- 支持图片格式的检索输出
- 支持多种文档格式的检索: csv, markdown, html, pdf

### 1.3 [Streamlit - UI](#ui)

- 使用Streamlit库进行UI组件开发

### 1.4 [多LLM协同的体现](#more_llm)

- 生成文本摘要使用 **大语言模型LLM**
- 图片的内容识别使用 **多模态视觉模型 GPT4ALL**
- 生成检索器使用 **嵌入模型 Embedding LLM**
- 生成问答使用 **大语言模型LLM**

### 1.5 使用Cython工具uvloop异步框架提升性能

- Linux系统特性，提升小模型在个人电脑的运行效果

### 1.6 支持配置Agent的构建模式

- 可以在配置文件设定各种大模型
  - 目前支持: Ollama, Google, ChatGPT, 其他API LLM



## 2 WorkFlow

目前支持多模态模型的调用

- 数据处理 - LLM
  - 对数据项进行概括和总结, 方便向量存储和Retriever的寻找
    - 对于PDF文件，根据数据类型的不同进行提取。或对所有数据进行一个图片的综合
- 创建检索器 - Retrieval
- 生成向量数据 - VectorStore
- 问答 - LLM
  - 通过检索器寻找相似度高的摘要，将uuid传给向量数据库寻找完整源数据
  - 将源数据作为提示词，使用LLM回答问题




## x0 环境配置

- python=3.12.3 (uvloop==0.19.0, Linux)

```bash
export HNSWLIB_NO_NATIVE=1

sudo apt-get install -y build-essential python3-dev libfreetype6-dev libpng-dev pkg-config
```

## x1 更新日志

```
v0.200	2024/11/07	yyz	*Push
	My AI - 基于RAG的多模态Agent构建工具
--- --- --- ---
v0.100→V0.111 - 2024-06-30 - [yyz]
	v0.1最后迭代版本
```

## x2 Static

#### Extract Data Type

![Extract_Data](../assets/MyAI/Extract_Data.png)

#### Query_1

![Query_1](../assets/MyAI/Query_1.png)

#### Vision_1

![Vision_1](../assets/MyAI/Vision_1.png)

#### More_LLM

![Many_LLM.png](../assets/MyAI/Many_LLM.png)

#### UI

![UI](../assets/MyAI/UI.png)
