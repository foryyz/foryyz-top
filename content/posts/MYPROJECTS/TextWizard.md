---
title: 'TextWizard'
date: 2025-04-10T02:18:51+08:00
draft: false
tags: ['foryyz', 'AI']
---

> 项目启动时间：2025/04/10
>
> 最后更新时间：2025/04/10
>
> 当前版本：demo-0.1
>
> 作者：yyz

# Text Wizard

## 0 How To Beginning

该工具为CLI程序，执行命令如下：

```bash
# conda activate text-wizard

# --debug 开启调试模式
python word_formatter.py <Input-Docx-File.docx> <Output-Docx-File.docx> --debug
```



## 1 DOCX文件的自动排版

实现原理：将较大的原docx文件分割后(解决大模型上下文长度的问题)使用DeepSeek模型对内容进行推理识别，让模型按json格式输出后汇总到新的docx文件中
