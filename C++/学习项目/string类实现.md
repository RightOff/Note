
# 实现string类

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
