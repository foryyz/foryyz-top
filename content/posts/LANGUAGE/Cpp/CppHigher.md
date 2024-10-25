---
title: 'C++ - 高级编程'
date: 2024-10-22T22:03:40+08:00
draft: false
---

# Cpp - Higher

## 1 模板

### 1.1 函数模板

#### 语法:

```cpp
template<typename T>
void mySwap(T &a, T &b){
    T temp = a;
    a = b;
    b = temp;
}

int main(){
    int a  = 1;
    int b = 2;
    mySwap<int>(a,b); // 显式指定类型
    cout << a <<endl;
    cout << b <<endl;

    return 0;
}
```

#### 用法：

- 自动类型推导
- 显式指定类型

#### 注意事项：

- 自动推导类型必须类型一致
- 模板必须要确定T的数据类型

#### 案例：

```cpp
```

