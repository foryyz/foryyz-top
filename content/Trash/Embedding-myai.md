---
title: 'Embedding模型的使用'
date: 2024-06-06T09:57:08+08:00
draft: false
---
# 1 安装Ollama和Embedding模型

```bash
sudo pacman -S ollama
systemctl start ollama.service

ollama pull mofanke/acge_text_embedding
```

# 2 创建对应python环境

```bash
conda create -n RAGLLM
pip install -U langchain_community tiktoken langchain-openai langchainhub chromadb langchain
pip install sentence-transformers
pip install huggingface_hub
pip install ipywidgets
pip install unstructured
pip install sentencepiece
```

# 3 编写第一个RAG程序

参考链接：https://github.com/langchain-ai/rag-from-scratch/blob/main/rag_from_scratch_1_to_4.ipynb

汉化版本链接：https://techdiylife.github.io/blog/topic.html?category2=t07&blogid=0050

对于LangChain调用第三方API的方法：

```python
from langchain import OpenAI
llm=OpenAI(
    temperature=0.7,
    openai_api_key='sk-cXdcebxeIfU9f4tG7e558fF96c35419bB3D66f4e196e8128',
    openai_api_base='https://api.bianxieai.com/v1'
)
```

第三方API使用：

- **BASE_URL** - https://api.bianxieai.com/v1
- **API Key** - sk-cXdcebxeIfU9f4tG7e558fF96c35419bB3D66f4e196e8128
- **model_name** - gpt-3.5-turbo