---
title: 使用SWIG封装CUDA接口供Python调用
author: 林大豆
categories: memo
tags: 
- CUDA
- Python 
- SWIG 
description: 使用SWIG封装CUDA接口供Python调用，在cmake中完成相关配置。
---
# Python--> C++/CUDA 
在cmake项目中，由swig封装C++/CUDA供Python调用。
## swig封装步骤
```shell
# source file: example.c  
# interface file: example.i

swig -tcl -python example.i

gcc -fpic -c example.c example_wrap.c -I/usr/include/python2.7

ld -shared example.o example_wrap.o -o _example.so

```
思路：将C编译器gcc替换成CUDA编译器nvcc  
```shell
# source file: example.c  
# interface file: example.i

swig -tcl -python example.i

nvcc --compiler-options '-fPIC' -c example.cu example_wrap.c -I/usr/include/python2.7 -I/usr/local/include -I`pwd`

nvcc -shared example.o example_wrap.o -o _example.so

```
**注：`.cu`为CUDA源文件标识扩展符**

## cmake的支持
- [cmake对swig的支持](https://cmake.org/cmake/help/v3.2/module/UseSWIG.html)
- [cmake对CUDA的支持](https://devblogs.nvidia.com/building-cuda-applications-cmake/)

eg: CMakeLists.txt   
```shell
cmake_minimum_required(VERSION 3.10)
project(python_call_demo LANGUAGES CXX CUDA)
set(CMAKE_CXX_STANDARD 11)

set(CMAKE_CXX_FLAGS  ${CMAKE_CXX_FLAGS}
        -fPIC)

find_package(CUDA REQUIRED)
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}
        --compiler-options '-fPIC'
        --std c++11
)

FIND_PACKAGE(SWIG REQUIRED)
INCLUDE(${SWIG_USE_FILE})

FIND_PACKAGE(PythonLibs 2.7)
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})

SET(CMAKE_SWIG_FLAGS "")

set(DEMO_SWIG_I_FILE
        cuda_test.i)

SET_SOURCE_FILES_PROPERTIES(${DEMO_SWIG_I_FILE} PROPERTIES CPLUSPLUS ON)
SET_SOURCE_FILES_PROPERTIES(${DEMO_SWIG_I_FILE} PROPERTIES SWIG_FLAGS "-includeall")
SET_SOURCE_FILES_PROPERTIES(${DEMO_SWIG_I_FILE} PROPERTIES SWIG_FLAGS "-c++")
SET_SOURCE_FILES_PROPERTIES(${DEMO_SWIG_I_FILE} PROPERTIES COMPILE_FLAGS "-bla -c++")

set(DEMO_SOURCE_FILES
        cuda_test.cu
        )

SWIG_ADD_LIBRARY(cuda_test LANGUAGE python SOURCES ${DEMO_SOURCE_FILES} ${DEMO_SWIG_I_FILE})
SWIG_LINK_LIBRARIES(cuda_test ${PYTHON_LIBRARIES})
```
