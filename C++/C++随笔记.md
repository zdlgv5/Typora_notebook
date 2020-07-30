## C++随笔记

## １. 锁简介

boost::mutex提供了跨平台的锁操作，不允许多个线程同时访问共享资源，从而确保共享资源不被脏写。在本文中仅仅是介绍简单的两种锁，最高效的锁boost::mutex和区域锁boost::mutex::scoped_lock

**boost::mutex例子**

```
#include <boost/thread/mutex.hpp>

boost::mutex m_mutexAccessServiceManager;

void CSettingCenter::ClearPlatformServiceInfoCache()

{

m_mutexAccessServiceManager.lock();

m_mapAccessServiceManager.clear();

m_mutexAccessServiceManager.unlock();

}
```

区域锁boost::mutex::scoped_lock顾名思义就是在作用域内有效，当离开作用域自动释放锁，传递参数是锁。区域锁就是把锁封装到一个对象里面。锁的初始化放到构造函数，锁的释放放到析构函数。这样当锁离开作用域时，析构函数会自动释放锁。即使运行时抛出异常，由于析构函数仍然会自动运行，所以锁仍然能自动释放。**一个典型的区域锁.**

```
boost::mutex::scoped_lock lock(user_initialized_mutex_);
```

应用于有大量的return返回的代码，避免出现死锁的问题

参考链接



- https://blog.csdn.net/wishchin/article/details/52537205;
- https://blog.csdn.net/wishchin/article/details/12884387.

## 2. 琐碎

####  两种确定字符串中,字符数的方法:

```
int len1 = str1.size() // string

int len2 = strlen(charr1) // char
```

#### 什么是oop?

Object oriented programming 面向对象编程。

####  什么是gp?

Generic programming 泛型编程。

#### 程序的编译流程

文本写代码（源代码）-->编译成机器语言（object code）-->将目标代码和其他代码链接起来，例如库-->可执行代码。

#### 数据类型碎片

1. #define 是c语言中的编译指令，被遗留进了C++，而在C++中，有更好的创建符号常量的办法：const。而在C++的头文件中，还是必须使用#define；

2. 变量的初始化

例如：

int a = 1; //C/C++

int a(1);  //C++

在C++中，初始化方式有了新的：

例如：

int emus = {7}; //初始化emus为7

int emus{7}; //初始化emus为7

int emus{}; //初始化emus为0

这种方式有助于更好的防范类型转换的错误。

3. 冷知识 unsigned 是unsigned int的缩写，主要优点是增加变量的存储最大值。

4. 当超过数据类型范围时候，其值将从取值范围另一端取值，但是C++不保证上溢和下溢时候不出错。

5. 选择数据类型的时候，要根据数据的大小选择，从而保证可移植性。

6. 使用const关键字进行常量声明的时候，应立即声明，分两行的时候，该常量会处在一种不确定的状态，造成错误。

7. const比define好的地方在于，const明确了常量的类型，const还可以用于更复杂的类型，例如数组和结构。

8. 浮点数的来源是基准值加上缩放因子，也就是移动小数点，从而称之为浮点。C++内部是2的次幂，而不是10次幂。浮点数可以表示的数可大可小。

   9.书写浮点数有两种方法，第一种就是正常的0.123；第二种是利用e，如3.45E6表示3.45*10的6次方，也就是1后面有6个0.6被称之为指数，3.45称之为尾数。E表示法适合非常大的数或者非常小的数。在书写中不能存在空格。

10. d.dddE+n表示将小数点向后移动n位,d.dddE-n表示将小数点向前移动n位。