---
title: 'LanceDB - 向量数据库的基本使用方法'
date: 2024-06-06T09:55:38+08:00
draft: false
tags: ['AI','LanceDB']
---
# LanceDB - 向量数据库

**安装** `pip install lancedb`

# 1 基础使用

## 1.1 创建客户端

```python
import lancedb

#创建客户端
url = "./data/sample-lancedb"
db = lancedb.connect(url)
```

## 1.2 创建表

```python
#1. 建表的同时添加数据
data = [
    {"vector": [3.1, 4.1], "item": "foo", "price": 10.0},
    {"vector": [5.9, 26.5], "item": "bar", "price": 20.0},
]
tbl = db.create_table("my_table", data=data)
#可选参数:
#mode='overwrite' - 覆盖已经创建的同名表
#exist_ok=True - 不覆盖已经创建的同名表，直接打开
```

```python
#2. 创建一张空表
schema = pa.schema([pa.field("vector", pa.list_(pa.float32(), list_size=2))])
tbl = db.create_table("empty_table", schema=schema)

# 直接添加数据
data = [
    {"vector": [1.3, 1.4], "item": "fizz", "price": 100.0},
    {"vector": [9.5, 56.2], "item": "buzz", "price": 200.0},
]
tbl.add(data)

# 添加df数据帧
df = pd.DataFrame(data)
tbl.add(data)
```

## 1.3 查找数据

```python
# Synchronous client

#通过向量来查找相似的向量
tbl.search([100, 100]).limit(2).to_pandas()

"""默认情况下没有对向量创建索引，因此是全表暴力检索。官方推荐数据量超过50万以上才需要创建索引，否则全表暴力检索的延迟也在可以接受的范围之内。"""
```

## 1.4 删除数据

```python
#类似SQL语法中的WHERE声明，需要指定字段和对应的值
tbl.delete('item = "fizz"')
```

## 1.5 修改数据

```python
#类似SQL语法中的UPDATE声明，需要指定字段和对应的值
table.update(where='item = "fizz"', values={"vector": [10, 10]})
```

## 1.6 删除表

```python
db.drop_table("my_table")
```

## 1.7 删除所有表

```python
print(db.table_names())
tbl = db.open_table("my_table")
```

