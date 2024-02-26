# c++基础

## class 和struct的区别

在C语言中struct是一种数据类型，不能包含任何函数。面向过程的编程认为，数据和操作是分离开的。但是到了面向对象的c++时，struct得到了很大的扩充，其可以包括成员函数、可以实现继承、可以实现多态。

那么在C++中struct和class的主要区别有：

+ 默认的继承访问权限。class默认的是private，struct默认的是public。在C++中class可以继承struct，struct也可以继承class。
+ 默认访问权限。struct作为数据结构的实现体，默认的数据访问控制是public，而class作为对象的实现体，默认的成员变量访问控制是private。
+ class关键字可以用于定义模板参数，例如 `template<class T>等效与template<typename T>`，而struct不能。
  > 综和以上三点，C++中struct更适合看成是一个数据结构的实现体，class更适合看成是一个对象的实现体。
  >
+ 在使用大括号上的区别。两者如果定义了构造函数的话，都不能用大括号进行初始化。如果没有定义构造函数，且所有成员变量全是public的话，都可以用大括号初始化。
  > PS：因此也不算是区别，知识struct成员默认是public，所以更多情况下都可以用大括号初始化。
  >

两者最大的区别就在于思想上，C语言编程单位是函数，语句是程序的基本单元。而C++语言的编程单位是类。从C到C++设计由以过程设计为中心向以数据组织为中心转移。

虽然感觉struct是多余的，但是考虑到“对C兼容性”，就将struct保留了下来，并做了一些扩展使其更适合面向对象，因此C++中的struct也不再是C中的struct。

## 输入流

+ `cin >> ch`会忽略任何空白字符（空格、制表符、换行符），`cin.get(ch)`读取每个字符以及空白字符。
+ 在命令行中，需要手动输入 EOF 来终止输入。在 Windows 上，可以通过按 `Ctrl+Z` 然后回车来发送 EOF；在 Unix/Linux/Mac 系统上，可以通过按 `Ctrl+D` 来发送 EOF。

## 编译

```
g++ -o hello hello.cpp	//编译输出一个可执行文件
```

`-o hello`选项告诉编译器将输出（一个可执行文件）命名为"hello"，没有错误将不会产生任何输出。

```
./hello		//执行可执行文件
```

## 指向指针的引用

```
int v = 1;
int *p = &v;
int *&rp = p;
```

`&`说明rp是一个引用。`*`确定rp引用的类型是一个指针。

## 关键字

### explicit

```
#include <iostream>
using namespace std;

class Point {
public:
    int x, y;
    Point(int x = 0, int y = 0)
        : x(x), y(y) {}
};

void displayPoint(const Point& p) 
{
    cout << "(" << p.x << "," 
         << p.y << ")" << endl;
}

int main()
{
    //如果构造函数不加explicit，发生隐式转换，输出(1,0)。 如果加上，则会报错：不存在从int转换到point的适当构造函数
    displayPoint(1);
    Point p = 1;
}
```

# C++练手项目

## 实现string类

1.创建自定义命名空间

2.确定基本框架

> size_t 类型和 int 的不同：
>
> 1. `size_t `在32位架构中被普遍定义为：`typedef unsigned int size_t`，为4字节。在64位架构中被定义为：`typedef unsigned long size_t`，为8字节。
> 2. int在不同架构下都是4字节，且int为带符号数，而size_t为无符号数。

> npos是什么？
>
> string::npos参数——npos是一个静态成员常量值，，用来表示不存在的位置。置npos=-1，但是由于size_是无符号整型，因此npos会转换成size_t的上限，也就是4294967295，在使用string时，我们的_size不可能到达这个数，最多到达4294967294，因此npos表示“不存在的位置”。

3.完善功能

3.1.重载[]，（两种重载，可读可写和只读）

3.2.构造函数与析构函数

> 成员函数形参列表后边的冒号用来初始化类的成员变量。

> 释放数组 delete 后加 []，`delete[] arr;`

3.3.str迭代器（两种，可读可写和只读）

3.4.拷贝（重难点）

3.5.赋值

3.6.reserve

3.7.resize

3.8.push_back

3.9.append(尾插字符串)

3.10insert（插入字符、插入字符串）

3.11.erase

3.12.c_str

3.13.size() 和 capacity()

3.14.运算符重载+=

3.15.重载运算符+、-

## WebServer


# 未解答的疑问

## C++primer书

### 练习4.21（p135）

```
vector<int> v1 = { 1,2,3,4,5,6,7,8,9 };
it = v1.begin();
while (it != v1.end())
{
	(*it) % 2 == 1 ? *it++ *= 2 : *it++;	//冒号后边不能为it++，为什么？


}
```

## 自定义命名空间

### 重定义错误

在其中写类没问题，但在其中写函数基本都会报重定义错误，如下图。除非在每个函数前边加上inline修饰。为什么？

![1704868904609](image/C++/1704868904609.png)

# Ubuntu相关命令

taskkill /im rdpclip.exe /f
