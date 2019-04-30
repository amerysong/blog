---
title: 函数模板
date: 2019-04-28 22:45:10
tags:
  - cs
  - lang
  - cpp
  - template
categories:
  - cs
  - lang
  - cpp
  - cpp templates:the complete guide
---

##  初识函数模板

### 定义函数模板

函数模板是参数化的函数，是一系列功能相同的函数的抽象，你可以使用不同的模板参数来调用它。如下是一个返回两个值中最大值的模板函数：

```c++
template <typename T>
T max (T a, T b)
{
    return b < a? a : b;
}
```

该模板函数定义了一系列返回两个值中最大值的函数，其参数**a**和**b**的类型由模板参数**T**指定。模板函数必须使用关键字**template**声明，模板参数位于`<>`中，以逗号分隔，如下所示：

> template<comma-separated-list-of-parameters\>

在我们的例子中，参数列表为**typename T**，其中，关键字**typename**引入类型参数，**T**为引入的类型参数，由调用者决定其具体类型，这里的类型参数**T**可以使用任何字母代替，这是目前C++程序中最常见的模板参数，但是其他参数也是允许的，参考 {% post_link nonetype-template-parameters 非类型模板参数 %} 一节了解更多信息。这个例子中，类型**T**必须支持`<`操作符，因为代码中**a**和**b**的比较使用了它，同时，类型**T**必须是可复制的，因为函数返回了它。

由于历史的原因，定义类型参数时你也可以使用**class**关键字来代替**typename**，**typename**关键字是C++98之后的标准引入的，在此之前，只能使用**class**来引入类型参数。因此，`max()`函数也可以使用以下的方式定义：

```c++
template <class T>
T max(T a, T b)
{
    return b < a? a : b;
}
```

在此上下文中，两种定义方式没有区别，但是***class***关键字可能会误导你认为这里**T**必须为一个类，因此建议使用***typename***以避免误导。

### 使用模板

以下程序展示了如何使用`max()`模板函数：

```c++
#include "max1.hpp"
#include <iostream>
#include <string>

int main()
{
	int i = 42;
	std::cout << "max(7, i):" << ::max(7, i) << std::endl;
	
	double f1 = 3.4;
	double f2 = 5.9;
	std::cout << "max(f1, f2):" << ::max(f1, f2) << std::endl;
	
	std::string s1 = "chinese";
	std::string s2 = "english";
	std::cout << "max(s1, s2):" << ::max(s1, s2) << std::endl;
}
```

程序中分别使用**int**、**double**和**std::string**三种类型的变量调用了三次`max()`函数，程序输出如下：

> max(7, i):42
> max(f1, f2):5.9
> max(s1, s2):english

`max()`函数调用时使用`::`来确保调用的是我们自己实现的函数而非标准库中的`std::max()`。编译器会根据模板函数的调用情况，分别为不同的类型生成相应的函数，我们的代码使用三种类型来调用`max()`函数，因此编译器会分别为三个类型分别生成相应的函数，即我们的调用生成了以下三个函数：

```c++
int max(int a, int b);
double max(double a, double b);
std::string max(std::string a, std::string b);
```

将类型参数（T）替换为实际类型（int、double、std::string）的过程称为模板实例化。

### 模板两阶段翻译

如果模板实例化使用的参数不支持模板函数体内的运算，这将会导致编译错误，例如以下代码将会编译出错：

```c++
#include "max1.hpp"
#include <complex>
int main()
{
    std::complex<float> c1, c2;  // std::complex不支持“<”运算
    ::max(c1, c2);
    return 0;
}
```

因此模板的编译过程分为两个阶段：

1. 在模板代码未实例化的定义期，对模板代码在忽略模板参数的前提下进行检查，这些检查包括：
   - 检查语法错误，如缺少分号
   - 使用不依赖模板参数的未知名称会被发现
   - 不依赖模板参数的静态断言会被检查
2. 在实例化阶段，模板代码会被再次检查以确保所有代码的合法性。也就是说，所有依赖模板参数的部分会被二次检查。

例如，以下代码：

```c++
template <typename T>
void foo(T t)
{
    undeclared(); // 第一阶段编译错误，如果undeclared()未知
    undeclared(t); // 第二阶段编译错误，如果undeclared(T)未知
    static_assert(sizeof(int) > 10, "int too small"); // 总是编译错误
    static_assert(sizeof(T) > 10, "T to small"); //如果T小于10，编译错误 
}
```

注意，一些编译器在第一阶段不会作全面检查，因此，在第二阶段前你不会马上看到编译错误。参考 {% post_link instantiation 模板实例化 %} 一节了解更多模板实例化内容。同时，两阶段翻译会导致一些问题：当函数模板以触发实例化的方式使用（使用模板参数和函数参数调用）时，编译器需要知道模板定义。这将会打破普通函数通常意义上的编译和链接的界限，因为编译器只需要知道普通函数的声明即可正确编译程序，参考  {% post_link template-in-practice 模板实例化 %} 一节了解更多关于该问题的内容。
