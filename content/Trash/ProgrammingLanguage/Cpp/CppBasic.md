---
title: 'C++ 1 - 基础语法'
date: 2024-06-22T16:34:00+08:00
draft: false
tags: ['C++']
---

# 一、认识C++

## 1.1 HelloWorld程序

```c++
#include <iostream>
using namespace std;

int main()
{
    cout << "hello world" << endl;
    
    system("pause");
    
    return 0;
}
```

## 1.2 注释

```c++
//单行注释

/*
多行
注释
*/
```

## 1.3 变量

**作用：**给一段指定的内存空间起名

```c++
//数据类型 变量名 = 变量初始值
int a = 1;
```

## 1.4 常量

**作用：**不可更改的数据

```c++
//方法1. #define 常量名 常量值（在文件上方定义）
//方法2. const 数据类型 常量名 常量值
#define Day 7
const int day = 7;
```

## 1.5 标识符命名规则

- 标识符**不能是关键字**
- 标识符**只能由字母、数字、下划线组成**
- 第一个字符**必须为字母或下划线**
- 标识符中字母**区分大小写**

# 二、数据类型

C++规定在创建一个变量或者常量时，必须要指定对应的数据类型，否则无法给变量分配内存

**意义：**给变量分配合适的内存空间

## 2.1 整型

| 数据类型            | 占用空间                                       | 取值范围                             |
| ------------------- | ---------------------------------------------- | ------------------------------------ |
| short(短整型)       | 2字节                                          | (-2^15 ~ 2^15-1) \| (-32768 ~ 32767) |
| int(整形)           | 4字节                                          | (-2^31 ~ 2^31-1)                     |
| long(长整型)        | windows 4字节 \| Linux 4字节(32位) 8字节(64位) | (-2^31 ~ 2^31-1)                     |
| long long(长长整型) | 8字节                                          | (-2^63 ~ 2^63-1)                     |

```c++
short num1 = 10;
int num2 = 10;
long num3 = 10;
long long num4 = 10;
```

## 2.2 sizeof

**作用：**统计数据类型所占内存大小

**语法：**sizeof(数据类型/变量)

```c++
int a = 10;
sizeof(int);
sizeof(a);
```

## 2.3 实型 (浮点型)

| 数据类型 | 占用空间 | 有效数字范围    |
| -------- | -------- | --------------- |
| float    | 4字节    | 7位有效数字     |
| double   | 8字节    | 15~16位有效数字 |

```c++
//默认情况下输出小数会显示6位有效数字
float f1 = 3.14f;//写f是为了减少一次转换
double d1 = 3.14;
//科学计数法
float f2 = 3e2;//e后正数表示 3*10^2
float f3 = 3e-2;//e后负数表示 3*0.1^2
```

## 2.4字符型

**作用：**字符型变量用于显示单个字符

- 一个字符型变量占用一个字节内存
- 字符串变量不是把字符本身放到内存中存储，而是将对应的ASCII编码放入到储存单元

```c++
//必须用单引号'',而且只能有一个字符
char ch = 'a';

/*ASCII编码
A - 65
a - 97
*/
```

## 2.5转义字符

**作用：**表示不能显示出来的ASCII字符

| 转义字符 | 含义                                                     |
| -------- | -------------------------------------------------------- |
| **\n**   | **换行**                                                 |
| **\t**   | **水平制表** (跳到下一个TAB位置),**一个制表符占8个位置** |
| \\\      | 代表一个反斜杠'\\'                                       |

## 2.6 字符串型

**作用：**表示一串字符

```c++
//C风格字符串
char str1[] = "hello world";
cout << str1 << end1;

//C++风格字符串
//包含一个头文件 #include <string>
string str2 = "hello world";
cout << str2;
```

## 2.7 布尔类型 bool

- true - 1 - 真
- false - 0 - 假

**bool类型占1个字节大小**

````c++
bool flag = true;
cout << flag;
```
    1//输出为 1
````

## 2.8 数据的输入

**作用：从键盘获取数据**

**关键字：cin**

**语法：**`cin >> 变量`

```c++
//举例
int a = 0;
cin >> a;

//cin 布尔类型数据时  只要不是0 就是1
```

# 三、运算符

- **算数运算符**
- **赋值运算符**
- **比较运算符** - 返回真假
- **逻辑运算符** - 返回真假

## 3.1 算数运算符

| 运算符     | 作用                                | 举例                      |
| ---------- | ----------------------------------- | ------------------------- |
| %          | 求模(取余) - 两个小数不能做取模运算 | 10 % 3 = 1 , 10 % 20 = 10 |
| +，-，*，/ | 加, 减, 乘, 除                      |                           |
| ++         | 递增                                |                           |
| --         | 递减                                |                           |

**前置(++a) 先++再计算表达式,后置(a++)先计算表达式再++**

## 3.2 赋值运算符

**+=,-=,*=,/=,%=**

## 3.3 比较运算符

| 运算符 | 作用           |
| ------ | -------------- |
| ==     | 相等于         |
| !=     | 不等于         |
| <, <=  | 小于, 小于等于 |
| >, >=  | 大于, 大于等于 |

## 3.4 逻辑运算符

**作用：**根据表达式的值返回真假

| 运算符 | 术语 | 作用                       |
| ------ | ---- | -------------------------- |
| !      | 非   | !a,若a真则为假,若a假则为真 |
| &&     | 与   | 全真为真,否则为假          |
| \|\|   | 或   | 有真即真,全假为假          |

# 四、程序流程结构

C/C++支持最基本的三种程序运行结构：**顺序结构、选择结构、循环结构**

## 4.1 选择结构

### 4.1.1 if语句

**if - else if - else**

```c++
int a = 0;
cin >> a;

if(a>20){
    cout << "分数大于20" << endl;
}
else if(a>50){
    cout << "分数大于50" << endl;
}
else{
   cout << "分数小于20" << endl; 
}
```

### 4.1.2 三目运算符

**语法：**`表达式1 ？ 表达式2 ： 表达式3`

**意义：**如果**1为真** 则**执行2** **否则执行3**

```c++
//返回更大的值
max = (a>b ? a : b);
//返回变量赋值
(a>b ? a : b) = 100;
```

### 4.1.3 switch语句

**执行多条件分支语句**

```c++
int score = 0;
cin >> score;
switch(score){
    case 1:
        cout << "score = 1" << endl;
        break;
    default:
        cout << "score = 0" << endl;
        break;
}
```

## 4.2 循环结构

### 4.2.1 while循环语句

```c++
while(true){
    cout << "这是一个死循环" << endl;
}
```

### 4.2.2 do...while循环语句

```c++
do{
    cout << "至少执行一次循环" << endl;
}while(true);
```

### 4.2.3 for循环语句

**语法：**for(**0**起始表达式;**1**条件表达式;**3**末尾循环体){**2**代码块}

```c++
//执行顺序 0123123123
for(int i=0; i<10; i++){
	cout << "i = " << i << endl;
}
```

### 4.2.4 嵌套循环

 **经典案例 - 打印乘法口诀表**

## 4.3 跳转语句

### 4.3.1 break 语句

### 4.3.2 continue 语句

### 4.3.3 goto 语句

```c++
goto FLAG:

FLAG:
```

# 五、数组

## 5.1 概述

所谓数组，就是一个**集合**，里面存放了**相同类型的数据元素**

**特点1**：数组中**每个数据元素**都是**相同的数据类型**
**特点2**：数组是由**连续的内存位置**组成的

## 5.2 一维数组



```c++
//三种定义方式

//数据类型 数组名[数组长度];
int array[3];//默认以随机数补齐

//数据类型 数组名[数组长度] = {值1,值2,..};
//......注:如果初始化时数据个数不足数组长度,会以0补齐
int array[10] = {0};//以0补齐

//数据类型 数组名[] = {值1,值2,..};

```

### 5.2.1 数组名的用途

**数组的地址就是数组第一个元素的地址**

**数组名是常量,不可以进行赋值操作**

1. 统计**整个数组在内存中的长度** - sizeof(arr)
2. 可以获取**数组在内存中的首地址** - cout << arr << endl;

### 5.2.2 冒泡排序

```c++
int arr[] = { 1,3,2,5,4 };
int length = sizeof(arr) / sizeof(arr[0]);
for (int i = 0; i <length-1 ; i++)
{
	for (int j = 0; j < length-i-1; j++)
	{
		if (arr[j]>arr[j+1])
		{
			int temp = arr[j + 1];
			arr[j + 1] = arr[j];
			arr[j] = temp;
		}
	}
}
for (int i = 0; i < length; i++)
{
	cout << arr[i] << endl;
}
```

## 5.3 二维数组

```c++
//四种定义方式
int arr1[2][3];
int arr2[2][3] = {{1,2,3},{4,5,6}};
int arr3[2][3] = {1,2,3,4,5,6};
int arr4[][3] = {1,2,3,4,5,6};
//遍历二维数组的方式
for(int i = 0 ; i < 2 ; i++){
    for(int j = 0 ; j < 3 ; j++){
        cout << arr[i][j] << endl;
    }
}
```

### 5.3.1 二维数组名的用途

```c++
//查看占用内存大小
cout << sizeof(arr) << endl;
//查看某一行占用内存大小
cout << sizeof(arr[0]) << endl;
//查看一个元素内存
cout << sizeof(arr[0][0]) << endl;

//求行数
cout << sizeof(arr) / sizeof(arr[0]);
//求列数
cout << sizeof(arr[0]) / sizeof(arr[0][0]);

//内存首地址
cout << arr << endl;
```

# 六、函数

## 6.1 定义

```
返回值类型 函数名(形式参数列表){
	函数体语句
	
	return表达式
}
```

## 6.2 调用

```
函数名(实际参数列表);
```

## 6.3 值传递

**值传递时,形参发生的任何改变,不会影响实参**

### 6.3.2 地址传递

**指针章节 7.7**

## 6.4 函数的声明

**声明可以多次,但定义只能有一次**

**语法：**`返回值类型 函数名(形式参数列表);`

## 6.5 分文件编写

1. 创建头文件 .h
2. 创建源文件 .cpp
3. 在头文件中写函数的声明
4. 在源文件中写函数的定义
5. 在源文件中 导入头文件 `#include "[headfilename].h"`

# 七、指针

## 7.1 概念

**指针的作用：**通过指针间接访问内存

- 内存编号从0开始记录,一般用十六进制数字表示
- 可以利用指针变量保存地址

**定义指针的语法：**`数据类型* 指针名;`

**解引用指针的语法：**`*指针名`   **作用：**找到指针指向内存中的数据

```c++
//定义指针
int a = 10;
int* p = &a;

//使用指针
*p = 20;
```

## 7.2 指针所占内存空间

- 32位 - 4个字节
- 64位 - 8个字节

## 7.3 空指针

**空指针：**指向内存编号为0的空间

**用途：**初始化指针变量

**注意：**空指针指向的内存不可以访问

**定义方式：**`数据类型* 指针名 = NULL;`

**0~255之间的内存编号是系统占用的，因此不可以访问**

## 7.4 野指针

**野指针：**指针指向非法的内存空间

```c++
//例 - 不知道的内存地址
int* p = (int*)0x1100;
```

## 7.5 const修饰指针

1. const修饰指针 - 常量指针
2. const修饰常量 - 指针常量
3. const即修饰指针，又修饰常量

```c++
int a = 10;
int b = 20;
//1.const修饰指针 - 常量指针
//指针的指向不能改
const int* p = &a;

//2.const修饰常量 - 指针常量
//指针指向的值不能改
int* const p = &a;

//3.const即修饰指针又修饰常量
const int* const p = &a;
```

## 7.6 指针访问数组

```c++
//用指针遍历数组
int a[] = { 0,1,2,3,4,5 };
int len = sizeof(a) / sizeof(a[0]);
int* p = a;
for (int i = 0; i < len; i++)
{
	cout << *p << endl;
	p++;
}
```

## 7.7 指针和函数



```c++
//交换函数程序
int main() {
	int a = 10;
	int b = 20;
	int* p1 = &a;
	int* p2 = &b;
	swap(p1, p2);

	cout << "a=" << a << " b=" << b << endl;
	//分割
	system("pause");
	return 0;
}

void swap(int* a, int* b) {
	int temp = *a;
	*a = *b;
	*b = temp;
}
```

# 八、结构体

## 8.1 定义与使用

**结构体即为自定义数据类型**

**语法：**`struct 结构体名{结构体成员列表};`

```c++
//定义结构体
struct Student{
    //成员列表
    string name;
    int age;
}s1 = {"张三",18};//可以定义结构体时顺便创建实体
//创建实体
Student li = {"李四",20};
```

## 8.2 结构体数组

**语法：**`struct 结构体名 数组名[元素个数] = { {} , {} ...};`

```c++
//创建结构体数组
Student stuArray[2] = {
{"张三",18},
{"李四",20}
};
//给结构体数组中的元素赋值
stuArray[1].name = "王五";
```

## 8.3 结构体指针

**作用：**通过指针访问结构体成员

通过操作符 `->` 访问

```c++
//例
Student zhang = {"yyz",21};
//指针类型要和结构体一样
Student* p = &zhang;
//通过指针访问
p->name;
```

## 8.4 结构体嵌套结构体

**结构体中的成员可以是另一个结构体,用来解决实际问题**

```c++
struct teacher{
    string name;
    Student stu;//辅导的学生
};
//创建老师实体
Student zhang = { "yyz",21 };
teacher johoho = { "焦红红",zhang};//学生实体为zhang
cout << johoho.stu.age;
```

## 8.5 结构体做函数参数

- **值传递** - 复制一份数据
- **地址传递** - 传递原数据

```c++
//例
#include <iostream>
#include <string>
using namespace std;

struct Student {
	string name;
	int age;
};

void fakechangestuname(Student stu);
void truechangestuname(Student* stu);

int main() {
	Student zhang = { "yyz",21 };
	cout << "初始名字为" << zhang.name;
	fakechangestuname(zhang);

	cout << "值传递后名字为" << zhang.name << endl;

	truechangestuname(&zhang);
	cout << "地址传递后名字为" << zhang.name << endl;
	//分割
	system("pause");
	return 0;
}

void fakechangestuname(Student stu) {
	stu.name = "www";
	cout << "值传递修改名字为" << stu.name << endl;
	cout << "执行成功" << endl;
}

void truechangestuname(Student* stu) {
	stu->name = "xxx";
	cout << "地址传递修改名字为" << stu->name << endl;
	cout << "执行成功" << endl;
}
```

## 8.6 结构体中const使用场景

**作用：**用const来防止误操作

```c++
//场景1.传递结构体时为了节省内存,一般采用地址传递,那为了防止修改原数据,要是用const修饰
//即在函数形参中添加const
void printname(const student* s){
    cout << s->name << endl;
}
```

## 8.7 结构体案例

```c++
//老师学生嵌套结构体实例
#include <iostream>
#include <string>
#include <ctime>
using namespace std;

struct Student {
	//成员列表
	string name;
	int score;
};

struct Teacher {
	string name;
	Student stulist[5];
};

void allocateSpace(Teacher tArray[], int len);
void printInfo(Teacher tArray[], int len);

int main() {
	//随机数种子
	srand((unsigned int)time(NULL));
	Teacher tArray[3];
	int len = sizeof(tArray) / sizeof(tArray[0]);
	allocateSpace(tArray, len);
	printInfo(tArray, len);

	//分割
	system("pause");
	return 0;
}

void allocateSpace(Teacher tArray[], int len) {
	string nameSeed = "ABCDEFGHIJKLMN";
	for (int i = 0; i < len; i++)
	{
		tArray[i].name = "Teacher_";
		tArray[i].name += nameSeed[i];
		for (int j = 0; j < 5; j++)
		{
			tArray[i].stulist[j].name = "Student_";
			tArray[i].stulist[j].name += nameSeed[j];
			int random = rand() % 61 + 40;
			tArray[i].stulist[j].score = random;
		}
	}
}

void printInfo(Teacher tArray[], int len) {
	for (int i = 0; i < len; i++)
	{
		cout << tArray[i].name << "的学生包括: " << endl;
		for (int j = 0; j < 5; j++)
		{
			cout << "\t" << tArray[i].stulist[j].name << "\t" << tArray[i].stulist[j].score << endl;
		}
	}
}
```

```c++
//冒泡排序 与 结构体
#include <iostream>
#include <string>
using namespace std;

struct Hero {
	//成员列表
	string name;
	int age;
	string sex;
};

void bubbleSort(Hero heroArray[], int len);

int main() {
	Hero heroArray[5] = {
		{"刘备",23,"男"},
		{"关羽",22,"男"},
		{"张飞",20,"男"},
		{"赵云",21,"男"},
		{"貂蝉",19,"女"}
	};
	int len = sizeof(heroArray) / sizeof(heroArray[0]);
	bubbleSort(heroArray, len);

	for (int i = 0; i < len; i++)
	{
		cout << heroArray[i].name << "\t";
	}

	//分割
	system("pause");
	return 0;
}

void bubbleSort(Hero heroArray[], int len) {
	for (int i = 0; i < len-1; i++)
	{
		for (int j = 0; j < len-i-1; j++)
		{
			if (heroArray[j].age > heroArray[j + 1].age) {
				Hero temp = heroArray[j];
				heroArray[j]= heroArray[j + 1];
				heroArray[j + 1] = temp;
			}
		}
	}
}
```



# 其他

```c++
//随机数种子
#include <ctime>
srand((unsigned int)time(NULL));
int a = rand()%100+1;//1-100的随机数字
```

# 经典案例

## 1. 水仙花数

```c++
#include <iostream>
using namespace std;

int main() {
	int num = 100;
	int a = 0;
	int b = 0;
	int c = 0;

	do {
		a = num % 10;
		b = num / 10 % 10;
		c = num / 100;
		if (a * a * a + b * b * b + c * c * c == num) {
			cout << num << "是水仙花数" << endl;
		}
		num++;
	} while (num <= 999);

	system("pause");
	return 0;
}
```

## 2. 打印乘法口诀表

```c++
for (int i = 1; i < 10; i++) {

	for (int j = 1; j < i+1; j++) {
		cout << i << " * " << j << " = " << i * j << "\t";
	}
	cout << endl;
}
```

## 3. 数组元素逆置

```c++
int arr[] = {1,3,2,5,4}
for (int i=0 ; i<sizeof(arr) / sizeof(arr[0]) ; i++){
    cout << arr[i] << endl;
}
int start = 0;//起始元素下标
int end = sizeof(arr) / sizeof(arr[0]) - 1;//末尾元素下标
int temp = arr[start];//第三只手
while(start < end){
    temp = arr[start];
    arr[start] = arr[end];
    arr[end] = temp;
    start++;
    end--;
}
cout << "逆置完成!" << endl;
for (int i=0 ; i<sizeof(arr) / sizeof(arr[0]) ; i++){
    cout << arr[i] << endl;
}
```

## 4.冒泡排序

**数组章节 5.2.2**
