---
title: 'MyAI - 个性化模型构建工具'
date: 2024-06-22T22:41:20+08:00
draft: false
tags: ['AI','foryyz']
---
> 项目启动时间：2024/06/02
>
> 最后更新时间：2024/06/30
>
> 当前版本：V0.111

# Hello My AI

项目名称：Hello My AI - 基于RAG和多LLM协作的现实系统构建工具

核心成员：待定

核心技术图示：https://github.com/langchain-ai/rag-from-scratch



# 0 更新日志

```
a0.001 - 2024-06-02 - [yyz]
核心算法实现,本地ollama模型和第三方LLM调取.

V0.100 - 2024-06-05 - [yyz]
核心算法 模块化,设计前端代码框架.(PyQT)

V0.101 - 2024-06-06 - [yyz]
前端代码框架优化,前后端逻辑实现.前端主窗口MainWindow类模块化.
- 后续前端代码暂由[JYY]接管.

V0.102 - 2024-06-09 - [yyz] [田歌]
实现Embedding向量数据的存储与读取.

V0.110 - 2024-06-11 - [yyz]
MainWindow窗口样式改动,按钮自动垂直排序
重构前端程序架构;重构模型处理与返回架构.
实现多检索 单大语言模型 分层处理 - 核心功能.
实现多向量数据库的分层设计，存储和读取测试成功.
添加utils.py文件，需要初始化的实例对象写入该文件中.
===下一步开发说明：编写 自动化添加[知识库登入按钮]，实现 [多补充文档数据的管理], 优化[ModelOne.py]代码结构.

V0.111 - 2024-06-30 - [yyz]
重构utils.py文件架构，改为面向对象编程结构
更改结构调用昵称(调用方式未改动)
- 尝试使用Spark集群训练Embedding模型，效果一般
- 目前改用Ollama的mxbai-embed-large模型
```



# 1 具体实现

## 1.1 开发路线：

第一阶段：核心算法实现，双前端设计，基础架构完毕。产出效果尚可。

- RAG模型检索、LLM模型输出、输入文档优化
  - 输入文档优化方式不唯一：
    - 过渡：使用第三方大模型处理 或 使用python脚本处理
    - 最终：使用本地大模型处理（需要解决速度问题） 或 使用国产第三方大模型处理（安全）
  - RAG检索方式不唯一：
    - 过渡：使用**Huggingface**平台开源Embedding模型
    - 优化：通过微调模型 或 调整参数的形式，提高检索质量
    - 其他方案：如果时间和技术能力允许，可以训练开发自己的Embedding模型
  - 输入文档的优化方案：
    - 暂定为 - 统一转化为json（用大模型处理效果很好）

预计开发时间 - 2024年8月中，互联网+大赛前

取得成绩后，继续优化参加后续比赛，如2025年计算机设计大赛 人工智能类 等。

如在互联网+大赛时未取得任何成绩或成绩非常不理想，**视情况终止项目或解散团队**。

## 1.2 技术栈

- 前端：PyQt, OpenWebUI
- 后端：不使用框架开发
- 向量数据库：Chroma(暂定)
- 版本管理：Git
- LLM框架：Ollama

## 1.3 变量说明 - utils.py

> 2024-06-30

- aigcs - 知识库实例列表



# x1 如何开始？

**操作系统**：**Ubuntu24.04** - WSL

## 0 安装Anaconda、Ollama

**Anaconda安装** - [Anaconda的安装与基本使用 | 🫨 Here is yyz! (foryyz.top)](https://foryyz.top/posts/othersite/howtouseanaconda/#0-安装anaconda)

**Ollama安装** - [Ollama的安装与基本使用 | 🫨 Here is yyz! (foryyz.top)](https://foryyz.top/posts/othersite/howtouseollama/#0-安装ollama)

## 1 使用Ollama拉取Embedding模型

```bash
#启动Ollama后执行 ↓
ollama pull mxbai-embed-large
```

## 2 创建Conda环境

```bash
conda create -n ragllm python=3.11
```

## 3 安装LangChain环境

```bash
#在执行下列命令前可以为pip配置国内源
pip install -U langchain_community tiktoken langchain-openai langchainhub chromadb langchain
pip install sentence-transformers
pip install huggingface_hub
pip install ipywidgets
pip install unstructured
pip install sentencepiece
```

## 4 安装PyQT环境

```bash
pip install pyqt6 pyqt6-tools
```

## 5 安装依赖包

```bash
sudo apt-get install libxkbcommon0 libegl1 libxcb-icccm4 libxkbcommon-x11-0 libxcb-image0 libxcb-keysyms1 libxcb-render-util0

#如果还是不能执行程序 请添加环境变量 export QT_DEBUG_PLUGINS=1 开启报错日志
```

## 6 拉取并执行程序

```bash
git pull https://github.com/foryyz/Hello-My-AI.git
cd Hello-My-AI

python run_GUI.py
```

