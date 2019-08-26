---
title: google breakpad备忘录
author: 林大豆
categories: memo
tags:
- memo
description: google breakpad备忘录
---

# 编译

```bash
git clone https://github.com/google/breakpad.git
or
git clone https://chromium.googlesource.com/breakpad/breakpad

cd breakpad

git clone https://chromium.googlesource.com/linux-syscall-support src/third_party/lss
```

# 在代码中添加exception handler
```c++
#include "client/linux/handler/exception_handler.h"

void exception_handler_init(const char * path) {
    google_breakpad::MinidumpDescriptor descriptor(path); // minidump文件写入到的目录
    static google_breakpad::ExceptionHandler eh(descriptor, NULL, NULL, NULL, true, -1);
}
```

# dump信息及解析

## 生成symbol信息
```bash
dump_syms ${path}/xxx.so > xxx.so.sym
```  
根据`head -n 1 xxx.so.sym`所得到的信息  
```
MODULE Linux x86_64 0B659766105832C2D9A83B22CBDD43480 xxx.so
```  
来创建目录并把symbol文件放到对应目录下  
```
mkdir -p symbols/xxx.so/0B659766105832C2D9A83B22CBDD43480
mv xxx.so.sym symbols/xxx.so/0B659766105832C2D9A83B22CBDD43480/
```

## 解析
```
minidump_stackwalk 4b5fc485-1c23-4bef-1700b383-3d47f73c.dmp ./symbols
```

# 参考资料
- [google-breakpad](https://chromium.googlesource.com/breakpad/breakpad/)
- [github-breakpad](https://github.com/google/breakpad)
- [Google Breakpad 学习笔记](https://www.jianshu.com/p/295ebf42b05b)
- [How To Add Breakpad To Your Linux Application](https://chromium.googlesource.com/breakpad/breakpad/+/master/docs/linux_starter_guide.md)
