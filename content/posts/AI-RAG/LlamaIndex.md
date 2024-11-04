---
title: 'LlamaIndex - Building a RAG pipeline'
date: 2024-11-03T14:18:00+08:00
draft: false
tags: ['AI','LLM','RAG']
---

# LlamaIndex - Building a RAG pipeline

## 1 **[Loading & Ingestion](https://docs.llamaindex.ai/en/stable/understanding/loading/loading/)**

1. Load the data
2. Transform the data
3. Index and store the data

### 1.1 Loaders

The way LlamaIndex does this is via data connectors, also called `Reader`

```python
#读取文件夹：
from llama_index.core import SimpleDirectoryReader
documents = SimpleDirectoryReader("./data").load_data()

#读取单个文档
from llama_index.core import Document
doc = Document(text="text")
```

### 1.2 Transformations

After the data is loaded, you then need to **process and transform your data** before putting it into a storage system. These transformations include **chunking**, **extracting metadata**, and **embedding each chunk**. This is necessary to make sure that the data can be retrieved, and used optimally by the LLM.

1. 处理和转换数据
2. 转换的方式包括 - 分块，提取元数据，嵌入每一个块

Transformation input/outputs are `Node` objects (a `Document` is a subclass of a `Node`). Transformations can also be stacked and reordered.

#### 1.2.1 High-Level Transformation API

Indexes have a `.from_documents()` method which accepts an array of Document objects

```python
# 使用.fron_documents() 接受 document数据并分块
from llama_index.core import VectorStoreIndex

vector_index = VectorStoreIndex.from_documents(documents)
vector_index.as_query_engine()
```

If you want to customize core components, like the text splitter, through this abstraction you can pass in a custom `transformations` list or apply to the global `Settings`

```python
from llama_index.core.node_parser import SentenceSplitter

text_splitter = SentenceSplitter(chunk_size=512, chunk_overlap=10)

# global
from llama_index.core import Settings

Settings.text_splitter = text_splitter

# per-index
index = VectorStoreIndex.from_documents(
    documents, transformations=[text_splitter]
)
```

#### 1.2.2 [Loading Data (Ingestion) - LlamaIndex](https://docs.llamaindex.ai/en/stable/understanding/loading/loading/#lower-level-transformation-api)

pass

### 1.3 Adding Metadata

You can also choose to add metadata to your documents and nodes.

```python
# 手动添加元数据
document = Document(
    text="text",
    metadata={"filename": "<doc_file_name>", "category": "<category>"},
)
```

### 1.4 Creating and passing Nodes directly

If you want to, you can create nodes directly and pass a list of Nodes directly to an indexer:

```python
# 手动添加node数据
from llama_index.core.schema import TextNode

node1 = TextNode(text="<text_chunk>", id_="<node_id>")
node2 = TextNode(text="<text_chunk>", id_="<node_id>")

index = VectorStoreIndex([node1, node2])
```



## 2 **[Indexing and Embedding](https://docs.llamaindex.ai/en/stable/understanding/indexing/indexing/)**

It's time to build an `Index` over these objects so you can start querying them.

### 2.1 What is  an Index

In LlamaIndex terms, an `Index` is a data structure composed of `Document` objects, designed to enable querying by an LLM. Your Index is designed to be complementary to your querying strategy.

下面只介绍两种常见的索引类型

#### 2.1.1 [Vector Store Index](https://docs.llamaindex.ai/en/stable/understanding/indexing/indexing/#vector-store-index)

The Vector Store Index takes your Documents and splits them up into Nodes. It then creates `vector embeddings` of the text of every node, ready to be queried by an LLM.

#### 2.1.2 [What is an embedding?](https://docs.llamaindex.ai/en/stable/understanding/indexing/indexing/#what-is-an-embedding)

`Vector embeddings` are central to how LLM applications function.

```
向量嵌入，通常简称为嵌入，是文本语义或含义的数字表示。具有相似含义的两段文本在数学上将具有相似的嵌入，即使实际文本完全不同。
```

**By default LlamaIndex uses `text-embedding-ada-002`**, which is the default embedding used by OpenAI.

When you want to search your embeddings, your query is itself turned into a vector embedding, and then a mathematical operation is carried out by VectorStoreIndex to rank all the embeddings by how semantically similar they are to your query.

```
原理：当您想要搜索嵌入向量时，您的查询本身将转换为向量嵌入向量，然后 VectorStoreIndex 执行数学运算，以根据它们与查询的语义相似程度对所有嵌入向量进行排序。
```

#### 2.1.3 [Top K Retrieval](https://docs.llamaindex.ai/en/stable/understanding/indexing/indexing/#top-k-retrieval)

pass

### 2.2 Using Vector Store Index

通过Documents列表创建

```python
# 要使用 Vector Store Index，请向其传递您在加载阶段创建的 Documents 列表
from llama_index.core import VectorStoreIndex

index = VectorStoreIndex.from_documents(documents)
#from_documents also takes an optional argument show_progress. Set it to True to display a progress bar during index construction.
# 可以添加show_progress=True参数来显示进度条
index = VectorStoreIndex.from_documents(documents,show_progress=True)
```

通过Node列表创建

```
from llama_index.core import VectorStoreIndex

index = VectorStoreIndex(nodes)
```

## 3 Storing

**Why?** By default, your indexed data is stored only in memory.

### 3.1 Persisting to disk

simplest way:

```python
# This works for any type of index.
index.storage_context.persist(persist_dir="<persist_dir>")
```

Composable Graph:

```python
graph.root_index.storage_context.persist(persist_dir="<persist_dir>")
```

loading:

```python
from llama_index.core import StorageContext, load_index_from_storage

# rebuild storage context
storage_context = StorageContext.from_defaults(persist_dir="<persist_dir>")

# load index
index = load_index_from_storage(storage_context)
```

### 3.2 Using Vector Stores

#### chroma

```python
pip install chromadb
```

steps:

- initialize the Chroma client
- create a Collection to store your data in Chroma
- assign Chroma as the `vector_store` in a `StorageContext`
- initialize your VectorStoreIndex using that StorageContext

Save:

```python
import chromadb
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import StorageContext

# load some documents
documents = SimpleDirectoryReader("./data").load_data()

# initialize client, setting path to save data
db = chromadb.PersistentClient(path="./chroma_db")

# create collection
chroma_collection = db.get_or_create_collection("quickstart")

# assign chroma as the vector_store to the context
vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# create your index
index = VectorStoreIndex.from_documents(
    documents, storage_context=storage_context
)

# create a query engine and query
query_engine = index.as_query_engine()
response = query_engine.query("What is the meaning of life?")
print(response)
```

Load:

```python
import chromadb
from llama_index.core import VectorStoreIndex
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import StorageContext

# initialize client
db = chromadb.PersistentClient(path="./chroma_db")

# get collection
chroma_collection = db.get_or_create_collection("quickstart")

# assign chroma as the vector_store to the context
vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# load your index from stored vectors
index = VectorStoreIndex.from_vector_store(
    vector_store, storage_context=storage_context
)

# create a query engine
query_engine = index.as_query_engine()
response = query_engine.query("What is llama2?")
print(response)
```

更多案例 [Chroma - LlamaIndex](https://docs.llamaindex.ai/en/stable/examples/vector_stores/ChromaIndexDemo/)

insert new documents:

```python
from llama_index.core import VectorStoreIndex

index = VectorStoreIndex([])
for doc in documents:
    index.insert(doc)
```

## 4 Querying

### 4.1 simplest:

```python
query_engine = index.as_query_engine()
response = query_engine.query(
    "Write an email to the user given their background information."
)
print(response)
```

### 4.2 Customizing the stages of querying

#### 4.2.1 设置tok_k的参数 并 添加后处理

use a different number for `top_k` and add a post-processing step that requires that the retrieved nodes reach a minimum similarity score to be included.

```python
from llama_index.core import VectorStoreIndex, get_response_synthesizer
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core.postprocessor import SimilarityPostprocessor

# build index
index = VectorStoreIndex.from_documents(documents)

# configure retriever
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=10,
)

# configure response synthesizer
response_synthesizer = get_response_synthesizer()

# assemble query engine
query_engine = RetrieverQueryEngine(
    retriever=retriever,
    response_synthesizer=response_synthesizer,
    node_postprocessors=[SimilarityPostprocessor(similarity_cutoff=0.7)],
)

# query
response = query_engine.query("What did the author do growing up?")
print(response)
```

#### 4.2.2 Configuring retriever

```python
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=10,
)
```

配置后处理器 [Configuring node postprocessors](https://docs.llamaindex.ai/en/stable/understanding/querying/querying/#configuring-node-postprocessors)

配置响应输出[Configuring response synthesis](https://docs.llamaindex.ai/en/stable/understanding/querying/querying/#configuring-response-synthesis)



## 相关文献：

[LlamaIndex](https://docs.llamaindex.ai/en/stable/)

[Building an LLM Application - LlamaIndex](https://docs.llamaindex.ai/en/stable/understanding/)
