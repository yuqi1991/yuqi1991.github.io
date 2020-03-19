---
title: C++11 Lambda 小结
date: 2018-06-12 03:43:37
tags:
- C++
- code
---


以前在使用python中常遇到使用lambda函数的需求，近日在使用C++11新增的lambda特性时发现其使用方法跟python存在不少差异。
   
 <!-- more -->
   lambda基本形式如下:
   **[ captures ] ( params ) -> ret { body }**


1. 关于捕获列表(captures): C++ 中的lambda需要在中括号里面声明捕获上下文的范围。大致的使用方法就不说了，但有一个细节，假如需要在类中引入某个类私有成员变量时，可设置捕获列表为“this”来引入，也可以使用“=”或“&”来隐式引入，前者为值传递，后者为引用传递。

2. 假如需要通过引用某个变量到函数中进行修改，在参数列表后面需要加上mutable，注意值传递和引用传递的差异。

3. 把lambda函数作为另一个函数的参数传入时(例如设置回调函数)，可使用std::bind绑定函数对象和参数来传入。要注意主体函数和lambda函数的作用范围， 因为假如主体函数所在类和lambda的函数所在类有同名的成员，会优先把主函数所在类的成员作为参数传入。

例子：

```C++
#include <iostream>
#include <functional>

class A{
public:
    int value = 0;
};

class B{
public:
    void foo(std::function<void(void)> _callback){  //C++14后支持参数列表中使用auto
        _callback();
    };
};

int main(void){
    A a;
    B b;
    b.foo(std::bind([&]() mutable {a.value++;}));  //为能修改value，此处加入mutable和使用引用传递
    std::cout << a.value << std::endl;
    return 0;
}
```

最后输出1