---
title: 'TextWizard - 字符操纵'
date: 2025-04-10T02:18:51+08:00
draft: false
tags: ['AI','Python','foryyz']
---

> 项目启动时间：2025/04/10
>
> 最后更新时间：2025/04/20
>
> 作者：yyz

# Text Wizard

​	字符操纵 - 大模型驱动的多个功能API组成的字符操纵集成工具

​	**开发日志及使用说明前往（未开源）：**https://github.com/foryyz/TextWizard

## 1 Docx Formatter

​	**基于DeepSeek-r1:70B 的 企业级 Docx文件的自动排版**
理论不限制最大文档长度，最新版本Demo.213成功完成了**20万字**的文档排版

#### `**实现思路**

​	**将较大的原docx文件分割后(解决大模型上下文长度的问题)使用DeepSeek模型对内容进行推理识别，让模型按json格式输出后汇总到新的docx文件中**

#### `支持格式

- 段落
- 标题
- 表格
- 图片 - Png, jpeg
  - 目前不支持对Smart图形的处理，但会在对应位置生成标注

#### `难点

1. **docx文件是流式文件**; 分割文档时，使用常规方法**按字数分隔会导致漏字 漏格式**(底层XML被切割) **漏内容**的问题。且图片和表格的重新排版难度高
2. **大模型本身能力不足**导致很难准确辨别每一个标题的层级，尤其是文件结构内容复杂时，判断极容易出错，仅凭优化提示词提升效果一般
   - 在前后文太短时，它缕不清先后高低，如可能将 1.1和1.1.1 算作同一级标题
   - 在前后文太长时，它会遗漏数据，如1.1.1和1.1.2之间的所有文字直接全部丢失
3. 编程语言库对word文档的支持较弱，对于格式敏感的文档 很难实现规范排版
4. png/jpeg常规格式图片的提取及Smart图形的提取方式不同，尤其是Smart图形是直接写于xml文件的，直接通过底层提取难度极高

#### `解决原理

1. 通过判断内容是否为段落的算法，将**段落分批传递**处理。同时通过判断内容是否为表格或图片的算法，尽可能将原样式原封不动传递。
2. 为各种标题设立各自的正则表达式，**只有常规程序算法检测不了的结构才交给模型解决**。
   为模型设计**记忆算法**，**将前部分的json标题总结作为记忆传递给模型**。
   将常规的文字Chunk分隔法修改为分段落处理，**为每个段落单独调用大模型**
   添加**后检测算法**，对模型的输出进行二次检验
3. 自写格式应用函数，通过config文件的方式统一管理，提高使用效率
4. 通过zipfile将word的媒体文件提取到固定目录，通过标记区分图片和图形，对于图片直接放置在原位，对于smart图形则添加警告标记

##### `失败的解决思路

- 先将docx转成pdf，通过**对每一页**进行读取识别，实现真正的按页拆分
  - X！**切割后表格会变得不连续**，排版会变的混乱



## 2 ProgressEyes

​	**进度的监控、判断、可视化及建议，适配各类WorkFlow软件**

### 如何开始：

MySQL 80

```sql
# 创建数据库text_wizard
# 创建数据表
CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nickname VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    priority INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

开始运行

```bash
streamlit run progress_eyes.py
```



技术栈：
	MySQL、StreamLit



## 3 AI SUB QUESTIONS

### 如何开始

MYSQL 80

```sql
CREATE TABLE ai_sub_questions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    question TEXT NOT NULL,
    standard_answer TEXT NOT NULL
);
```

开始运行

```bash
streamlit run ai_sub_questions.py
```

