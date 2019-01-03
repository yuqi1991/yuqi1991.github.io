---
title: Gtest 测试框架使用方法整理
date: 2018-07-10 21:58:00
tags:
- C++
- unit test
- gtest
---

  最近面试的几家公司都有关于嵌入式unit test框架的问题，想到自己从没有正儿八经地学习过这类型的测试框架，久闻google的gtest大名，于是拉下来在clion上学着试用了一下。
   <!-- more -->

gtest源代码可以在[https://github.com/google/googletest]()直接pull下来，里面已经除了gtest以外还有一个mock框架-gmock，用于生成虚拟类进行打桩测试。
  
  


###   关于在CLion的使用配置


 
gtest根目录下已经有配置好了的CMakeList.txt文件，在clion上使用只需要把整个gtest文件夹复制到项目目录中，再通过配置项目本身的CMakeList文件即可启用gtest。


CMakeList.txt配置例子如下


```c++
cmake_minimum_required(VERSION 3.10)
project(c_lion_test)

add_subdirectory(./googletest)
include_directories(./googletest/googletest/include)
include_directories(./googletest/googlemock/include)

set(CMAKE_CXX_STANDARD 11)
set(LIBRARIES gtest gtest_main pthread)


add_executable(c_lion_test main.cpp)
target_link_libraries(c_lion_test ${LIBRARIES})
```


### 框架使用

待续...


  
  
  