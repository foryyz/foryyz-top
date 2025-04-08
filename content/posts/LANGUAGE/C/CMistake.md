---
title: 'C - MISTAKE'
date: 2025-04-08T23:03:44+08:00
draft: false
tags: ['C']
---

# C - MISTAKE



### 1 关系运算符的优先级相同

```C
int a;
scanf("%d", &a);

if (3<a<10){
  printf("T");
}else{
  // 这段代码永远不会走到这一步，比如输入 a=-2, 3<a返回0 由0又和10比较 返回1, 所以恒True
  printf("F");
}
```

### 2 sizeof 不是函数 而是一个运算符

sizeof() 是长度运算符，并不属于函数
