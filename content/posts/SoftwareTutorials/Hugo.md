---
title: 'HugoUse'
date: 2025-04-08T22:54:20+08:00
draft: false
tags: ['Hugo']
---

> Author - yyz
>
> Create Time - 2025/04/08
>
> **Last Update Time - 2025/04/08**

# HUGO

## 基础命令

```bash
# 查看版本
hugo version

# 初始化站点 - 在当前目录下创建新的Hugo站点
hugo new site <站点名>

# 创建新文章
hugo new <路径(posts/??)>/<文件名>.md

#编译生成静态文件并启动web服务
hugo server

# 编译生成静态文件 - 将编译所有文件并输出到public目录  
hugo

#版本和环境详细信息
hugo env
```

## 简单的密码功能

(静态博客只能实现伪加密，查看html源代码就能看到文章内容)

在`layouts/_default/single.html`的.Content段替换

```html
{{- if .Content }}
<div class="post-content">
  {{- if not (.Param "disableAnchoredHeadings") }}
  {{- partial "anchored_headings.html" .Content -}}
  {{- else }}{{ .Content }}{{ end }}
</div>
{{- end }}

替换为：

{{- $password := .Params.password | default "" -}}
{{- if .Content }}
<div class="post-content" id="post-content" style="display:none;">
  {{- if not (.Param "disableAnchoredHeadings") }}
  {{- partial "anchored_headings.html" .Content -}}
  {{- else }}{{ .Content }}{{ end }}
</div>

{{- if ne $password "" }}
<script>
(function() {
    var input = prompt("请输入文章密码：");
    if (input === "{{ $password }}") {
        document.getElementById("post-content").style.display = "block";
    } else {
        alert("密码错误！");
        if (history.length <= 1) {
            window.location.href = "/";
        } else {
            history.back();
        }
    }
})();
</script>
{{- else }}
<script>
    // 没有设置密码的文章直接显示
    document.getElementById("post-content").style.display = "block";
</script>
{{- end }}
{{- end }}
```

之后只要在文章的头部加上 `password: xxxxx` 字段即可
