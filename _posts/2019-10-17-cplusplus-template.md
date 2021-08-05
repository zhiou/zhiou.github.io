---
priority: 0x6
title: C++11可变参数函数模板展开原理
excerpt: 从EOS源码看C++11新特性
categories: [Summarise]
background-image: climb.jpeg
tags:
  - C++11
  - 可变参数函数模板
---



C++11新特性中的可变参数模板，相对于原来的va_list展开方式，提供了递归和初始化列表展开两种方式，我们分别看下其中的原理。

### 递归方式展开

递归方式展开是比较容易理解的，只需要提供一个展开函数模板和终止函数，例如：

```c++
void expand() // 也可以是带固定个数参数的模板，以提前终止递归
{
  std::cout << "finished" << std::endl;
}

template <typename T, typename ...Args>
void expand(T arg, Args... rest)
{
  std::cout << arg << std::endl;
  expand(rest...);
}
```

根据终止函数的不同，可以设定递归的迭代次数。

### 初始化列表展开

借用初始化列表和逗号表达式的特性，也可以达成参数展开的目的。例如：

```c++
template<typename T>
void func(T t)
{
  std::cout << t << std::endl;
}

template<typename... Args>
void expand(Args... args)
{
  std::initializer_list<int>{(func(args), 0)...};
}
```

这里需要对```std::initializer_list<T>{(func(args), 0)...};```做下解释，（func(args), 0) 其实是利用逗号表达式的特性，先执行func(args), 然后返回0，而(func(args), 0)... 的作用就是变长参数展开。这里之所以需要使用逗号表达式，原因是func函数返回值为void， 而初始化列表是不支持void作为参数的，如果func函数本身有返回类型，假设为T func(T t), 那么展开表达式可以写成 ```std::initializer_list<T>{func(args)...};```.当然这样做没什么意义。

初始化列表展开相对递归展开来说，它只能一次性展开参数包里的所有参数。

