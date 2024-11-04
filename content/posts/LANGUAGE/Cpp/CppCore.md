---
title: 'C++ - 核心编程'
date: 2024-09-22T20:34:00+08:00
draft: false
tags: ['C++']
---

# 一、内存分区模型

C++程序执行时，内存大方向分为**4个区域**

- **代码区：**存放函数体的二进制代码
- **全局区：**存放**全局变量**和静态变量以及**常量**
- **栈区：**由**编译器自动分配释放**,存放函数的参数值,局部变量 - 特点：**先进后出**
- **堆区：**由**程序员分配和释放**,若不主动释放则在程序结束时由操作系统回收

## 1.1 程序运行前

编译后生成exe可执行程序,**未执行该程序前分为**两个区域

**代码区：**存放 CPU 执行的机器指令

- **共享** - 共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可
- **只读** - 防止程序意外地修改了它的指令

**全局区：**该区域的数据在程序结束后由操作系统释放

​	**包括：**全局变量、静态变量、字符串常量、常量区和其他常量

```c++
//静态变量
static int a = 10;
```

## 1.2 程序执行后

**栈区：**

​	由编译器自动分配释放, 存放函数的参数值,局部变量等

​	**注意事项：不要返回局部变量的地址，栈区开辟的数据由编译器自动释放**

**堆区：**

​	**由程序员分配释放**,若程序员不释放,程序结束时由操作系统回收

```c++
//使用new在堆区开辟内存
int* a = new int(10);
int* arr = new int[10];
```

## 1.3 new操作符

**语法：**`new 数据类型`

​	利用new创建的数据，会**返回**该数据对应的类型的**指针**

​	使用 `delete 指针 `释放堆区内存

```c++
//堆区开辟数组
int* func()
{
	int* arr = new int[10];
	return arr;
}
```



# 二、引用

**作用： **给变量起别名

**语法：** `数据类型 &别名 = 原名`

```c++
//a的别名b
int a = 10;
int& b = a;
```

**引用可以引用栈区和堆区，不能引用全局区的数据**

```c++
//int& c = 10; //这行代码会报错，访问非法内存空间
```

## 2.1 引用注意事项

- 引用**必须初始化**
- 引用在**初始化后，不可以改变**

```c++
//int &a; 这种定义方式是错误的
const int & ref = 10;//不会报错，本质是 int temp = 10;int& ref = temp;只不过temp是编译器临时定义的，我们找不到它
```

## 2.2 引用做参数传递

**作用：**函数传参时，可以利用引用的技术**让形参修饰实参**

**优点：**可以简化指针**修改实参**

```c++
//交换函数的 引用传递 实现方式
void swap(int& a, int& b) {
	int temp = a;
	a = b;
	b = temp;
}

int main() {
	int a = 1;
	int b = 2;
	swap(a, b);
	cout << a << endl;
	cout << b << endl;
	//分割
	system("pause");
	return 0;
}
```

> 通过引用参数产生的效果同按地址传递是一样的。引用的语法更清楚简单

## 2.3 引用做函数返回值

- **1.引用可以作为函数的返回值**
- **2.如果函数的返回值是引用，则函数的调用可以作为左值**

**注意：不要返回局部变量引用**

```c++
//1.
int& IntReturn(){
    static int a = 10;
    return a;
}
//2.
int &ref = IntReturn();//即 ref 引用 变量a
IntReturn() = 1000;//即 a 的值变为1000
```

## 2.4 引用的本质

**引用在C++内部实现就是一个指针常量**(指向不可以更改，指向的值可以更改)

```c++
int a = 10;
int& ref = a;
//本质是 int* const ref = &a;

cout << ref << endl;
//本质是 cout << *ref << endl; 编译器自动做了 解引用 的操作
```

## 2.5 常量引用

**作用：**常量引用主要**用来修饰形参，防止误操作**。

```c++
//在函数形参列表中，可以加const修饰形参，防止形参改变实参
void showValue(const int& v){
    //v += 10; //用来防止 这一行 发生,如果没有const修饰并执行了这行代码，则实参也被修改了
	cout << v << endl;
}
```



# 三、函数提高

## 3.1 函数默认参数

```c++
//在声明函数时设置默认参数
int func(int a=1, int b=2, int c=3);

int func(int a, int b, int c){
    return a+b+c;
}
```

## 3.2 函数占位参数

**补：占位参数也可以有默认值**

```c++
//例
void func(int a, int){
	cout << "占位参数" << endl;
}

//func(1)//这样是非法的
func(1,2);//这个2是用不到的
```

## 3.3 函数重载

### 3.3.1 概述

**作用：**函数名可以相同，提高复用性

**函数重载满足条件：**

* 同一个作用域下
* 函数名称相同
* 函数**参数** **类型不同**  或者 **个数不同** 或者 **顺序不同**

**注意:**  函数的返回值不可以作为函数重载的条件

```c++
void func(){
    cout << "函数没有接收到参数" << endl;
}
void func(int a){
    cout << "函数接收到整型参数" << a << endl;
}
void func(double b, int a){
    cout << "函数接收到双浮点型参数" << b << endl;
    cout << "函数接收到整型参数" << a << endl;
}
```

### 3.3.2 注意事项

* 引用作为重载条件

```c++
void func(int &a){
    cout << "func (int &a) 调用" << endl;
}

void func(const int &a){
    cout << "func (int &a) 调用" << endl;
}

int a = 10;
func(a);
func(10)
```



# 四、类和对象*

**C++面向对象** 三大特征：**封装、继承、多态**

**万事万物皆为对象，对象上有其属性和行为**

具有相同性质的**对象**，抽象称为**类**

类中的属性和行为 统一称为**成员**
属性也称为 - 成员属性 成员变量
行为也称为 - 成员方法 成员函数

**语法：** `class 类名{ 访问权限:  属性 / 行为 };`

## 4.1 封装

### 4.1.1 封装的意义

- 将属性和行为作为一个整体，表现生活中的事物
- 将属性和行为加以权限控制

### 4.1.2 封装的权限控制

- **public - 公共权限** - 成员 类内可以访问 类外也可以访问
- **protected - 保护权限** - 成员 类内可以访问 类外不可以访问，可以继承
- **private - 私有权限** - 成员 类内可以访问 类外不可以访问，不可以继承

### 4.1.3 struct 和 class区别

唯一的区别在于 **默认的访问权限不同**

- struct 默认权限为公共 public
- class 默认权限为私有 private

### 4.1.4 成员属性设置为私有

**优点1：**将所有成员属性设置为私有，可以**自己控制读写权限**

**优点2：**对于写权限，可以**检测数据的有效性**

```c++
//1.控制读写权限
class Person{
    int age;
public:
    void getAge(){ //只读权限
        return age;
    }
}

//2.判断数据的有效性
class Student{
    int score;
public:
    int setScore(int Score){ //只读权限
        if(score > 60){
            return age;
        }
        return 0;
    }
}
```

## 4.2 对象的初始化和清理

C++中的面向对象来源于生活，每个对象也都会有初始设置 以及 对象销毁前的清理数据的设置。

### 4.2.1 构造函数和析构函数

对象的**初始化和清理**也是两个非常重要的安全问题

​	1. 一个对象或者变量**没有初始状态**，对其使用后果是未知

​	2. 同样的使用完一个对象或变量，**没有及时清理**，也会造成一定的安全问题

c++利用了**构造函数**和**析构函数**解决上述问题，这两个函数将会**被编译器自动调用**，完成对象初始化和清理工作。

对象的初始化和清理工作是编译器强制要我们做的事情，因此如果**我们不提供构造和析构，编译器会提供**

**编译器提供的构造函数和析构函数是空实现。**

* 构造函数：主要作用在于创建对象时**为对象的成员属性赋值**，构造函数由编译器自动调用，无须手动调用。
* 析构函数：主要作用在于对象**销毁前**系统自动调用，执行一些清理工作。

#### 4.2.1.1 语法

**构造函数语法：**`类名(){}`

1. 构造函数，没有返回值也不写void
2. 函数名称与类名相同
3. **构造函数可以有参数，因此可以发生重载**
4. 程序在调用对象时候会自动调用构造，无须手动调用,而且**只会调用一次**

**析构函数语法：** `~类名(){}`

1. 析构函数，没有返回值也不写void
2. 函数名称与类名相同,在名称前加上符号  ~
3. **析构函数不可以有参数，因此不可以发生重载**
4. 程序在对象销毁前会自动调用析构，无须手动调用,而且**只会调用一次**

```c++
class Person
{
public:
	//构造函数
	Person()
	{
		cout << "Person的构造函数调用" << endl;
	}
	//析构函数
	~Person()
	{
		cout << "Person的析构函数调用" << endl;
	}

};
```

### 4.2.2 构造函数的分类及调用

两种分类方式：

​	按参数分为： 有参构造和无参构造

​	按类型分为： 普通构造和拷贝构造

三种调用方式：

​	括号法

​	显示法

​	隐式转换法

```c++
//拷贝构造函数 写法
	Person(const Person& p) {
		age = p.age;
		cout << "拷贝构造函数!" << endl;
	}

//注意：调用无参构造函数不能加括号，如果加了编译器认为这是一个函数声明
	//Person p2();
Person p; //调用无参构造函数

//注意：不能利用拷贝构造函数初始化匿名对象  编译器认为是对象声明
//Person(p4);

//2.2 显式法
Person p = Person(10); 
//Person(10)单独写就是匿名对象  当前 行结束之后，马上析构

//2.3 隐式转换法
Person p = 10; // Person p = Person(10); 
Person p5 = p4; // Person p5 = Person(p4);
```

#### 4.2.2.1 拷贝构造函数调用时机

C++中拷贝构造函数调用时机通常有三种情况

* 使用一个已经创建完毕的对象来初始化一个新对象
* 值传递的方式给函数参数传值 - 值传递的本质就是拷贝构造
* 以值方式返回局部对象

```c++
//1. 使用一个已经创建完毕的对象来初始化一个新对象
Person man(100); //p对象已经创建完毕
Person newman(man); //调用拷贝构造函数

//2. 值传递的方式给函数参数传值 - 在执行dowork函数时 新创建了一个临时p 即调用了拷贝构造函数
//相当于Person p1 = p;
void doWork(Person p1) {}
void test02() {
	Person p; //无参构造函数
	doWork(p);
}

//3. 以值方式返回局部对象
Person doWork2()
{
	Person p1;
	cout << (int *)&p1 << endl;//验证 不是同一个对象
	return p1;
}
void test03()
{
	Person p = doWork2();//doWork2函数返回了一个P1，将p1赋给p时 即调用了拷贝构造
	cout << (int *)&p << endl;//验证 不是同一个对象 和上面的地址不同
}

```

### 4.2.4 构造函数调用规则

默认情况下，c++编译器至少给一个类添加3个函数

1．默认构造函数(无参，函数体为空)

2．默认析构函数(无参，函数体为空)

3．默认拷贝构造函数，对属性进行值拷贝

**构造函数调用规则如下：**

* 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造


* 如果用户定义拷贝构造函数，c++不会再提供其他构造函数

### 4.2.5 深拷贝与浅拷贝

深浅拷贝是面试经典问题

**浅拷贝：**简单的赋值拷贝操作

**深拷贝：**在堆区重新申请空间，进行拷贝操作

**浅拷贝带来的问题是 堆区的内存重复释放 那么要利用深拷贝解决**

​	**析构函数执行时，也要将在堆区开辟的数据进行释放**



 ```c++
 class Person {
 public:
     m_height = new int(height);
    
 //深拷贝构造函数  
 Person(const Person& p) {
 	cout << "拷贝构造函数!" << endl;
 	//如果不利用深拷贝在堆区创建新内存，会导致浅拷贝带来的重复释放堆区问题
 	m_height = new int(*p.m_height);
 }
 
 //析构函数 释放的标准写法
 ~Person() {
 	cout << "析构函数!" << endl;
 	if (m_height != NULL)
 	{
 		delete m_height;
 	}
 }
 ```

### 4.2.6 初始化列表

**作用：**C++提供了初始化列表语法，用来初始化属性

**语法：**`构造函数()：属性1(值1),属性2（值2）... {}`

```c++
//传统方式初始化
Person(int a) {
    m_A = a;
    m_B = b;
	m_C = c;
}

//初始化列表方式初始化
//1.
Person() :m_A(1), m_B(2), m_C(2) { }
//2.
Person(int a, int b, int c) :m_A(a), m_B(b), m_C(c) {
    //函数体就没什么可以写了
}
```

### 4.2.7 类对象作为类成员

C++类中的成员可以是另一个类的对象，我们称该成员为 **对象成员**

```c++
//例 - B类中有对象A作为成员，A为对象成员
class A {}
class B
{
    A a；//a就是对象成员
}
```

**构造的顺序是 ：先调用对象成员的构造，再调用本类构造**
**析构顺序与构造顺序相反**

```c++
class Phone{
public:
    string m_PhoneName;
	Phone(string name){
		m_PhoneName = name;
		cout << "Phone构造" << endl;
	}
};

class Person{
public:
	string m_Name;
	Phone m_Phone;
	//初始化列表可以告诉编译器调用哪一个构造函数
	Person(string name, string pName) :m_Name(name), m_Phone(pName)//m_Phone(pName) 相当于是做了隐式转换法来调用构造函数Phone(string name)
    {
		cout << "Person构造" << endl;
	}
};

void test01()
{
	//当类中成员是其他类对象时，我们称该成员为 对象成员

	Person p("张三" , "苹果X");
	p.playGame();

}
```

### 4.2.8 静态成员

静态成员就是在成员变量和成员函数前加上关键字static，称为静态成员

静态成员分为：

*  静态成员变量
   *  **所有对象共享同一份数据**
   *  在编译阶段分配内存 - 全局区
   *  类内声明，类外初始化
*  静态成员函数
   *  所有对象共享同一个函数
   *  **静态成员函数只能访问静态成员变量**

**静态成员变量**

```c++
class Person{	
public:
    static int m_A; //静态成员变量 类内声明
private:
	static int m_B; //静态成员变量也是有访问权限的
};

int Person::m_A = 10;//类外初始化
int Person::m_B = 10;//类外初始化

//两种访问方式
//1、通过对象
Person p1;
p1.m_A = 100;
//2、通过类名
cout << "m_A = " << Person::m_A << endl;

//cout << "m_B = " << Person::m_B << endl; //私有权限访问不到
```

**静态成员函数**

```c++
class Person{
public:
	static void func(){
		cout << "func调用" << endl;
		m_A = 100;
		//m_B = 100; //错误，不可以访问非静态成员变量
	}
	static int m_A; //静态成员变量
	int m_B; // 
};

int Person::m_A = 10;//静态成员变量初始化

//静态成员变量两种访问方式
//1、通过对象
Person p1;
p1.func();

//2、通过类名
Person::func();
```

## 4.3 C++对象模型和this指针

### 4.3.1 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储

**只有非静态成员变量才属于类的对象上**

空对象占用内存空间为：1，是为了区分空对象占内存的位置

```c++
class Person {
public:
	Person() {
		mA = 0;
	}
	//非静态成员变量占对象空间
	int mA;
	//静态成员变量不占对象空间
	static int mB; 
	//函数也不占对象空间，所有函数共享一个函数实例
	void func() {
		cout << "mA:" << this->mA << endl;
	}
	//静态成员函数也不占对象空间
	static void sfunc() {
	}
};
```

### 4.3.2 this指针概念

**this指针的本质是一个指针常量，指针的指向不可修改**

**this指针指向被调用的成员函数所属的对象**

this指针的用途：

*  当形参和成员变量同名时，可用this指针来区分
*  在类的非静态成员函数中返回对象本身，可使用return *this



```c++
class Person
{
public:

	Person(int age)
	{
		//1、当形参和成员变量同名时，可用this指针来区分
		this->age = age;
	}

	Person& PersonAddPerson(Person p)//引用方式返回，是返回本体
	{
		this->age += p.age;
		//返回对象本身
		return *this;
	}

	int age;
};

void test01()
{
	Person p1(10);
	cout << "p1.age = " << p1.age << endl;

	Person p2(10);
	p2.PersonAddPerson(p1).PersonAddPerson(p1).PersonAddPerson(p1);
	cout << "p2.age = " << p2.age << endl;
}
```

### 4.3.3 空指针访问成员函数

C++中空指针也是可以调用成员函数的，但是也要注意有没有用到this指针

```c++
Person * p = NULL;
p->ShowClassName(); //空指针，可以调用成员函数
p->ShowPerson();  //但是如果成员函数中用到了this指针，就不可以了
```

如果用到this指针，需要加以判断保证代码的健壮性

```c++
void ShowPerson() {
	if (this == NULL) {
		return;
	}
	cout << mAge << endl;
}
```

### 4.3.4 const修饰成员函数

**常函数：**

* 成员函数后加const后我们称为这个函数为**常函数**
* **常函数内不可以修改成员属性**
* 成员属性声明时**加关键字mutable**后，在常函数中依然**可以修改**

**常对象：**

* 声明对象前加const称该对象为常对象
* 常对象只能调用常函数

```c++
class Person {
public:
	Person() {
		m_A = 0;
		m_B = 0;
	}
	//this指针的本质是一个指针常量，指针的指向不可修改 本质为Type* const pointer
	//如果想让指针指向的值也不可以修改，需要声明常函数
	void ShowPerson() const {//const的本质是 const this指针
		
		//this = NULL; //不能修改指针的指向 Person* const this;
		//this->mA = 100; //但是this指针指向的对象的数据是可以修改的

		//const修饰成员函数，表示指针指向的内存空间的数据不能修改，除了mutable修饰的变量
		this->m_B = 100;
	}

	void MyFunc() const {
		//mA = 10000;
	}
public:
	int m_A;
	mutable int m_B; //可修改 可变的
};

//const修饰对象  常对象
void test01() {
	const Person person; //常量对象  
	cout << person.m_A << endl;
	//person.mA = 100; //常对象不能修改成员变量的值,但是可以访问
	person.m_B = 100; //但是常对象可以修改mutable修饰成员变量

	//常对象访问成员函数
	person.MyFunc(); //常对象不能调用const的函数
}
```

## 4.4类外写成员函数

**::表示作用域**

```c++
class Person{
public:
    void func();
}

Person::func(){// ::表示作用域
    //函数体
}
```

## 4.5 友元

在程序里，有些私有(private)属性 也想让类外特殊的一些函数或者类进行访问，就需要用到友元的技术

**友元的目的就是让一个函数或者类 访问另一个类中私有成员**

**友元的关键字为 friend**

友元的三种实现

* 全局函数做友元
* 类做友元
* 成员函数做友元

### 4.5.1 全局函数做友元

```c++
class Person
{
	//在类中使用friend声明全局函数 就使类做友元了
	friend void func(Person * person);//在主函数中 这个函数就能访问私人权限属性了
};
```



### 4.5.2 类做友元

```c++
class Building;
class goodGay
{
public:
	goodGay();
	void visit();
private:
	Building *building;
};

class Building
{
	//告诉编译器 goodGay类是Building类的好朋友，可以访问到Building类中私有内容
	friend class goodGay;
public:
	Building();
public:
	string m_SittingRoom; //客厅
private:
	string m_BedRoom;//卧室
};
```

### 4.5.3 成员函数做友元

```c++
class goodGay{
public:
	void visit(){
        cout << "goodGay的成员函数" << endl;
    }
private:
	Building *building;
};

class Building{
	//告诉编译器  goodGay类中的visit成员函数 是Building好朋友，可以访问私有内容
	friend void goodGay::visit();
private:
    int a;//变量a的值 友元元素可以访问
}
```

## 4.6 运算符重载

- 加号运算符重载
- 左移运算符重载 - 只能在全局函数中实现
- 递增运算符重载
- 赋值运算符重载 - 也涉及 **深拷贝浅拷贝**的问题
- 关系运算符重载
- 函数调用运算符重载

```c++
class MyInteger {
private:
	int* m_num;

public:
	MyInteger() {
		this->m_num = new int(1);
		//cout << "*this->m_num = " << *this->m_num << endl;
		//cout << "this->m_num = " << this->m_num << endl;
	}
	MyInteger(int a) {
		this->m_num = new int(a);
	}

	MyInteger(const MyInteger& myInteger) {
		cout << "调用拷贝构造函数!" << endl;
		if (myInteger.m_num != nullptr){
			this->m_num = new int(*myInteger.m_num);
		}
		else {
			this->m_num = nullptr;
		}
	}

	~MyInteger(){
		if (this->m_num != NULL){
			cout << "已为值为" << *this->m_num << "的对象执行析构函数" << endl;
			delete this->m_num;
		}
	}

	int getter() {
		return *m_num;
	}
	int* getterAddress() {
		return m_num;
	}

	//加号运算符重载
	MyInteger operator+(MyInteger& myInteger) {
		MyInteger temp;
		if (this->m_num != nullptr && myInteger.m_num != nullptr){
			*temp.m_num = *this->m_num + *myInteger.m_num;
		}
		return temp;
	}

	//递增运算符重载 - 前置++ 先++，后返回
	MyInteger& operator++() {
		*m_num = *m_num + 1;
		return *this;
	}
	//递增运算符重载 - 后置++ 先返回，后++
	MyInteger operator++(int) {
		MyInteger temp = *this;
		*m_num = *m_num + 1;
		return temp;
	}

	//赋值运算符重载 - 如果类中有属性指向堆区，做赋值操作时也会出现深浅拷贝问题
	MyInteger& operator=(const MyInteger& myIngeter) {
		if (this->m_num != NULL) {
			//delete this->m_num;
			//this->m_num = NULL;
			*this->m_num = *myIngeter.m_num;
			return *this;
		}
		this->m_num = new int(*myIngeter.m_num);
		return *this;
	}

	//关系运算符 - ==
	bool operator==(const MyInteger& myIngeter) {
		if (*this->m_num == *myIngeter.m_num) {
			return true;
		}
		return false;
	}
	//关系运算符 - <=
	bool operator<=(const MyInteger& myIngeter) {
		if (*this->m_num <= *myIngeter.m_num) {
			return true;
		}
		return false;
	}
	//关系运算符 - >=
	bool operator>=(const MyInteger& myIngeter) {
		if (*this->m_num >= *myIngeter.m_num) {
			return true;
		}
		return false;
	}
	void operator()(string text) {
		cout << text << endl;
	}
};

//左移运算符重载 - 只能使用全局函数实现
ostream& operator<<(ostream& cout, MyInteger& myInt){
	cout << myInt.getter();
	return cout;
}

int main() {
	MyInteger int1;
	MyInteger int2(2);
	MyInteger()("i love u");//匿名函数调用
    cout << "++int1 = " << ++int1 << endl;
//	cout << "int1++ = " << int1++ << endl;
	cout << "int1 = " << int1.getter() << endl;
}
```

## 4.7 继承

**语法：** `class A : public B{};` 

A 类称为子类 或 派生类  ，  B 类称为父类 或 基类

### 4.7.1 继承方式

![cpp_img1](B:\MarkDown\Picture\cpp_img1.png)

>  父类中私有成员也是被子类继承下去了，只是由编译器给隐藏后访问不到

### 4.7.2 继承中构造和析构顺序

继承中 先调用父类构造函数，再调用子类构造函数，析构顺序与构造相反

### 4.7.3 继承同名成员处理方式

问题：当子类与父类出现同名的成员，如何通过子类对象，访问到子类或父类中同名的数据呢？

* 访问子类同名成员   直接访问即可
* 访问父类同名成员   需要加作用域

```c++
//instance
class Father{
public:
    int m_A;
};
class Son : public Father{
public:
    int m_A
};

int main(){
    Son s;
    s.m_A;
    s.Father::m_A;
}
```

### 4.7.4 继承同名静态成员处理方式

问题：**继承中同名的静态成员**在子类对象上如何进行访问？

静态成员和非静态成员出现同名，处理方式一致

- 访问子类同名成员   直接访问即可
- 访问父类同名成员   需要加作用域

### 4.7.5 多继承语法

C++允许**一个类继承多个类**

语法：` class 子类 ：继承方式 父类1 ， 继承方式 父类2...`

多继承可能会引发父类中有同名成员出现，需要加作用域区分

**C++实际开发中不建议用多继承**

### 4.7.6 菱形继承

**菱形继承概念：**

​	两个派生类继承同一个基类

​	又有某个类同时继承者两个派生类

​	这种继承被称为菱形继承，或者钻石继承

![cpp_img2](B:\MarkDown\Picture\cpp_img2.png)

**菱形继承问题：**

1.      羊继承了动物的数据，驼同样继承了动物的数据，当草泥马使用数据时，就会产生二义性。
2.      草泥马继承自动物的数据继承了两份，其实我们应该清楚，这份数据我们只需要一份就可以。



### 4.7.7 虚拟继承

**继承虚基类**又**通过虚基表指针** 与 **其指向的虚基表**来实现，这就是虚拟继承的三板斧。

- 一个类调用构造函数的顺序：**虚基类->直接基类->类**
- 虚基类只初始化一次

virtual base pointer - **vbptr**

virtual base table - vbt - 虚基表

[虚拟继承 讲解文章]: https://blog.csdn.net/weixin_44212838/article/details/125971934#comments_31364081

[虚拟继承 讲解视频]: https://www.bilibili.com/video/BV1et411b73Z?t=812.0&amp;p=134



## 4.8  多态

### 4.8.1 多态的基本概念

**多态是C++面向对象三大特性之一**

多态分为两类

* 静态多态: 函数重载 和 运算符重载属于静态多态，复用函数名
* **动态多态: 派生类和虚函数实现运行时多态**

静态多态和动态多态区别：

* 静态多态的函数地址早绑定  -  编译阶段确定函数地址
* **动态多态**的函数地址晚绑定  -  **运行阶段确定函数地址**

多态满足条件

* 有继承关系
* 子类重写父类中的虚函数

多态使用条件

* 父类指针或引用指向子类对象

```c++
class Animal{
public:
    virtual speak(){//Speak函数就是虚函数 编译器在编译的时候就不能确定函数调用
        cout << "Animal Speak!" << endl;
    }
}
class Cat : public Animal{
public:
	void speak(){
		cout << "小猫在说话" << endl;
	}
};
class Dog : public Animal{
public:
	void speak(){
		cout << "小狗在说话" << endl;
	}
};

void DoSpeak(Animal & animal){//多态使用条件：父类指针或引用指向子类对象
	animal.speak();
}

int main() {
    Cat cat;
	DoSpeak(cat);
    Dog dog;
	DoSpeak(dog);
    
	system("pause");
	return 0;
}
```

> 注：重写：函数返回值类型  函数名 参数列表 完全一致称为重写

### 4.8.2 纯虚函数和抽象类

**纯虚函数：** `virtual 返回值类型 函数名 （参数列表）= 0 ;`

当类中**有了纯虚函数**，这个类也称为**抽象类**

**抽象类特点**：

 * 无法实例化对象
 * 子类必须重写抽象类中的纯虚函数，否则也属于抽象类

```c++
class Base{
public:
	//类中只要有一个纯虚函数就称为抽象类
	//抽象类无法实例化对象
	//子类必须重写父类中的纯虚函数，否则也属于抽象类
	virtual void func() = 0;//纯虚函数
};
```

### 4.8.3 虚析构和纯虚析构

多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码

解决方式：将父类中的析构函数改为**虚析构**或者**纯虚析构**

虚析构和纯虚析构共性：

* 可以解决父类指针释放子类对象
* 都需要有具体的函数实现

虚析构和纯虚析构区别：

* 如果是纯虚析构，该类属于抽象类，无法实例化对象

虚析构语法：`virtual ~类名(){}`

纯虚析构语法：` virtual ~类名() = 0;`     `类名::~类名(){}`

```c++
class Animal {
public:
	Animal(){
		cout << "Animal 构造函数调用！" << endl;
	}
	virtual void Speak() = 0;

	//析构函数加上virtual关键字，变成虚析构函数
	//virtual ~Animal()
	//{
	//	cout << "Animal虚析构函数调用！" << endl;
	//}
	virtual ~Animal() = 0;
};

Animal::~Animal(){
	cout << "Animal 纯虚析构函数调用！" << endl;
}

//和包含普通纯虚函数的类一样，包含了纯虚析构函数的类也是一个抽象类。不能够被实例化。

class Cat : public Animal {
public:
	Cat(string name){
		cout << "Cat构造函数调用！" << endl;
		m_Name = new string(name);
	}
	virtual void Speak(){
		cout << *m_Name <<  "小猫在说话!" << endl;
	}
	~Cat(){
		cout << "Cat析构函数调用!" << endl;
		if (this->m_Name != NULL) {
			delete m_Name;
			m_Name = NULL;
		}
	}
public:
	string *m_Name;
};

void test01(){
	Animal *animal = new Cat("Tom");
	animal->Speak();

	//通过父类指针去释放，会导致子类对象可能清理不干净，造成内存泄漏
	//怎么解决？给基类增加一个虚析构函数
	//虚析构函数就是用来解决通过父类指针释放子类对象
	delete animal;
}

int main() {
	test01();
    
	system("pause");
	return 0;
}
```



**总结：**

​	1. 虚析构或纯虚析构就是用来解决通过父类指针释放子类对象

​	2. 如果子类中没有堆区数据，可以不写为虚析构或纯虚析构

	3. 拥有纯虚析构函数的类也属于抽象类



# 五、文件操作

程序运行时产生的数据都属于临时数据，程序一旦运行结束都会被释放

通过**文件可以将数据持久化**

- C++中对文件操作需要包含头文件 **<fstream >**

文件类型分为两种：

1. **文本文件**     -  文件以文本的**ASCII码**形式存储在计算机中
2. **二进制文件** -  文件以文本的**二进制**形式存储在计算机中，用户一般不能直接读懂它们

操作文件的三大类:

- ofstream：写操作
- ifstream： 读操作
- fstream ： 读写操作

## 5.1 文本文件的 读写

```c++
//读写文件步骤如下：

//1.包含头文件
#include <fstream>

//2.创建流对象
ofstream ofs;
ifstream ifs;
fstream fs;

//3.打开文件
ofs.open("文件路径",打开方式);

//4.读写数据 利用<<可以向文件中写数据
ofs << "写入的数据";

//5.关闭文件
ofs.close();
```

文件打开方式：

| 打开方式    | 解释                       |
| ----------- | -------------------------- |
| ios::in     | 为读文件而打开文件         |
| ios::out    | 为写文件而打开文件         |
| ios::ate    | 初始位置：文件尾           |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 二进制方式                 |

**注意：** 文件打开方式可以配合使用，利用|操作符

**例如：**用二进制方式写文件 `ios::binary |  ios:: out`

### 5.1.2读数据的方式

**示例：**

```C++
#include <fstream>
#include <string>
void test01()
{
	ifstream ifs;
	ifs.open("test.txt", ios::in);

	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
		return;
	}

	//第一种方式
	//char buf[1024] = { 0 };
	//while (ifs >> buf)
	//{
	//	cout << buf << endl;
	//}

	//第二种
	//char buf[1024] = { 0 };
	//while (ifs.getline(buf,sizeof(buf)))
	//{
	//	cout << buf << endl;
	//}

	//第三种
	//string buf;
	//while (getline(ifs, buf))
	//{
	//	cout << buf << endl;
	//}

	char c;
	while ((c = ifs.get()) != EOF)
	{
		cout << c;
	}

	ifs.close();


}

int main() {

	test01();

	system("pause");

	return 0;
}
```

## 5.2 二进制文件的 读写

以二进制的方式对文件进行读写操作

打开方式要指定为 ==ios::binary==

#### 5.2.1 写文件

二进制方式写文件主要利用流对象调用成员函数write

函数原型 ：`ostream& write(const char * buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数



**示例：**

```C++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];
	int m_Age;
};

//二进制文件  写文件
void test01()
{
	//1、包含头文件

	//2、创建输出流对象
	ofstream ofs("person.txt", ios::out | ios::binary);
	
	//3、打开文件
	//ofs.open("person.txt", ios::out | ios::binary);

	Person p = {"张三"  , 18};

	//4、写文件
	ofs.write((const char *)&p, sizeof(p));

	//5、关闭文件
	ofs.close();
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

总结：

* 文件输出流对象 可以通过write函数，以二进制方式写数据



#### 5.2.2 读文件

二进制方式读文件主要利用流对象调用成员函数read

函数原型：`istream& read(char *buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数

示例：

```C++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];
	int m_Age;
};

void test01()
{
	ifstream ifs("person.txt", ios::in | ios::binary);
	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
	}

	Person p;
	ifs.read((char *)&p, sizeof(p));

	cout << "姓名： " << p.m_Name << " 年龄： " << p.m_Age << endl;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

- 文件输入流对象 可以通过read函数，以二进制方式读数据
