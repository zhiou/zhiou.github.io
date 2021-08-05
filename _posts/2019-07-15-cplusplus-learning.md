---
priority: 0.6
title: C++11 features
excerpt: 从EOS源码看C++11新特性
categories: [Summarise]
background-image: climb.jpeg
tags:
  - C++11
  - EOS

---

1. std::function & std::bind

1. What does std::move & std::forward do?

   To be answered;

2. The differences of emplace_back & push_back.

   二者的区别在于他们的参数类型不同 :)。

   emplace_back 的参数列表为```Args&&... __args```, 传入的参数列表将直接通过std::forward 转发给相应的构造函数进行对象的构造.所以在emplace_back过程里，最多发生一次构造或者拷贝构造，并且emplace_back的参数既可以是元素类型构造所需要的参数，也可以是已经构造好的对象。

   而push_back的参数类型为值类型或者引用类型，只能传入对象或者其引用，当然如果对象支持隐式转换的情况下，也可以传入其构造函数所需的参数，但这会导致发生一次构造和拷贝构造。

   总的来说，二者在容器添加已存在的对象时其实是没有区别的，都只发生一次拷贝构造或者移动构造，但如果是添加新对象到容器，emplace_back 会是更好的选择，但前提是直接传入构造所需的参数，如果先构造了对象再使用emplace_back，那么是无法体现emplace_back的优势的。

   举个例子：

   ```c++
   class A {
     explicit A(int p):m(p) {}
     A(int p, int q)m:(p+q) {}
     private:
     int m;
   };
   
   int main(int argc, char* argv[])
   {
     vector<A> vec;
     vec.push_back(0);  //无法通过编译， A(int)要求显示调用，如果移除explicit关键字，可以编译，发生一次构造，一次拷贝构造
     vec.push_back(A(0)); // OK，一次构造，一次拷贝构造
     vec.emplace_back(0); // OK, 一次构造
     vec.emplace_back(A(0)); // OK, 一次构造，一次拷贝构造
     vec.push_back(1, 2); // 无法通过编译
     A a(1,2);
     vec.push_back(a); //OK,一次拷贝构造
     vec.emplace_back(a); //OK, 一次拷贝构造
     vec.emplace_back(1,2);// OK,一次构造
   }
   ```

   至于有人提出的内存重分配导致引用失效问题，其实是vector本身的特性，在元素个数超过vector容量时会发生内存重分配，那么对元素的引用当然会失效，这并非是emplace_back独有的。

   

3. What is std::enable_if used for?

   * 可以完成函数模板的参数类型条件检查

     例如	

   	 ```c++
   static inline auto has_field( F flags, E field )
      -> std::enable_if_t< std::is_integral_v<F> && std::is_unsigned_v<F> &&
                           std::is_enum_v<E> && std::is_same_v< F, std::underlying_type_t<E> >, bool>
      ```

   has_field 返回bool类型，条件是F是无符号整型，E是std::underlying_type_t的枚举值，且实际类型与F是一致的。

   * 也能用来实现返回值不同的函数重载。

     例如

     ```c++
     template <std::size_t k, class T, class... Ts>
     typename std::enable_if<k==0, typename element_type_holder<0, T, Ts...>::type&>::type
     get(tuple<T, Ts...> &t) {
       return t.tail; 
     }
     template <std::size_t k, class T, class... Ts>
   typename std::enable_if<k!=0, typename element_type_holder<k, T, Ts...>::type&>::type
     get(tuple<T, Ts...> &t) {
       tuple<Ts...> &base = t;
       return get<k-1>(base); 
     }
     ```
     
     get函数在k 是否为0时生效的模板是不同的，所以上面的定义不会发生冲突。

4. How does lamda expression capture variables?

Lambda 表达式利用[]捕获外部变量，并且可以指定捕获哪些变量，用什么方式

| 捕获列表  | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| `[a]`     | `a`为值传递                                                  |
| `[a, &b]` | `a`为值传递，`b`为引用传递                                   |
| `[&]`     | 所有变量都用引用传递。当前对象（即`this`指针）也用引用传递。 |
| `[=]`     | 所有变量都用值传递。当前对象用引用传递。                     |

> 注意，当用引用方式捕获外部变量时，需要关注被引用变量的生命周期，如果是临时变量可能由于超出生命周期已被释放，导致引用已经失效。
