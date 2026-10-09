Ubuntu 本身就是一个基于 Linux 的操作系统，其核心库（glibc）和系统调用与 C 语言标准有着天然的亲和性。你不需要像在 macOS 上那样通过 Docker 或交叉编译来寻找“纯粹”的环境，因为 Ubuntu 的原生环境本身就是 C 语言开发的标准参考平台之一。

## 搭建纯粹的ANSI C99环境
1.在终端执行以下命令，它会安装GCC编译器、GNU Make以及必要的标准C开发文件（libc-dev）.

```bash
sudo apt update
sudo apt install build-essential gdb
sudo apt install vim
```
2. 严格标准进行编译
```bash
gcc -std=c99 -pedantic -Wall -Wextra -Werror -g -O0 -o hello hello.c
```

`build-essential` 是一个元包（`meta-package`），它本身不包含文件，只是依赖其他包。所以查询它的路径需要分几个层面来看。查询`build-essential`依赖的包
```bash 
   apt-cache depends build-essential
```
```text
build-essential
 |Depends: libc6-dev
  Depends: <libc-dev>
    libc6-dev
  Depends: gcc
  Depends: g++
  Depends: make
    make-guile
  Depends: dpkg-dev
```
这说明 build-essential 只是把 gcc、make、libc6-dev 等包捆绑在一起，本身没有实际文件。
查看某个包安装的所有文件路径
```bash
dpkg -L gcc
dpkg -L g++
dpkg -L make
dpkg -L libc6-dev
dpkg -L dpkg-dev
```
查看gcc实际使用的头文件搜索路径
```bash
echo | gcc -E -v - 2>&1 | sed -n '/#include <...> search starts here:/,/End of search list./p'
```
查看gcc实际使用的库搜索路径
```bash
gcc -print-search-dirs
```

## 标准C头文件和GCC内部头文件的区别

### 标准C头文件
位置：/usr/include
特点：
- 由 C标准库（Ubuntu上是 glibc）提供
- 声明的是函数接口，这些函数的实现在动态库中（libc.so）
- 依赖操作系统（Linux系统调用）
- 安装包：libc6-dev
示例：
```c
// /usr/include/stdio.h
extern int printf(const char *format, ...);
extern int scanf(const char *format, ...);
// 这些函数的实现在 /lib/x86_64-linux-gnu/libc.so.6 中
```
### GCC内部头文件
位置：/usr/lib/gcc/x86_64-linux-gnu/<version>/include   
特点：
- 由 GCC编译器提供
- 定义的是编译器内置类型和宏，与操作系统无关
- 这些头文件的内容依赖编译器的具体实现
- 不需要链接任何库，纯粹是编译时的定义
示例：
```c
// /usr/lib/gcc/.../include/stddef.h（简化版）
typedef __SIZE_TYPE__ size_t;      // 编译器内置类型
typedef __PTRDIFF_TYPE__ ptrdiff_t;
#define NULL ((void*)0)
#define offsetof(type, member) __builtin_offsetof(type, member)
// 注意：__SIZE_TYPE__、__builtin_offsetof 都是GCC内置的
```

有的文件，如stdint.h在/usr/include中和/usr/lib/gcc/x86_64-linux-gnu/<version>/include中都有，因为gcc的stdint.h是一个包装器，它会根据情况包含glibc的版本：
```c
// /usr/lib/gcc/.../include/stdint.h（简化）
#ifndef _GCC_WRAP_STDINT_H
#define _GCC_WRAP_STDINT_H

#if __STDC_HOSTED__
    // 如果有操作系统，使用系统的 stdint.h
    #include_next <stdint.h>
#else
    // 如果裸机环境（如STM32），GCC自己定义
    typedef __INT8_TYPE__ int8_t;
    typedef __INT16_TYPE__ int16_t;
    // ...
#endif

#endif
```
#include_next 是 GCC 的扩展，意思是"从下一个搜索路径继续找"。这就是为什么 GCC 的头文件能"包装"系统头文件。

可以直接在终端中让GCC预处理并输出所有预定义的宏，然后查找它：
```bash
gcc -dM -E - < /dev/null | grep __STDC_HOSTED
gcc -dM -E - < /dev/null | grep __SIZE_TYPE__
```

写一个最小程序，观察size_t被替换成什么:
```c
//test.c
#include <stddef.h>
size_t a;
```
运行gcc -E test.c -o test.i，查看test.i文件，可以看到size_t被替换成了typedef long unsigned int size_t;

如果想看gcc编译器自己在哪里定义这个宏，需要查看gcc的C源码：
```bash
wget https://ftp.gnu.org/gnu/gcc/gcc-13.3.0/gcc-13.3.0.tar.gz
tar xzf gcc-13.3.0.tar.gz
cd gcc-13.3.0
```
```c
// gcc/c-family/c-cppbuiltin.cc
void
c_cpp_builtins (cpp_reader *pfile)
{
  // ...
  builtin_define_with_value ("__SIZE_TYPE__", SIZE_TYPE, 0);
  builtin_define_with_value ("__PTRDIFF_TYPE__", PTRDIFF_TYPE, 0);
  // ...
}

这里的 SIZE_TYPE 是一个字符串宏，它的值（如 "long unsigned int"）由 GCC 在配置和编译时根据目标平台决定，定义在 gcc/config/<arch>/<arch>.h 中。

例如，x86_64 平台的定义：
```c
// gcc/config/i386/i386.h
#define SIZE_TYPE (TARGET_64BIT ? "long unsigned int" : "unsigned int")
```
**核心概念**
```text
编译器启动
    ↓
根据目标平台（x86_64、ARM等）设置内置宏
    ↓
__SIZE_TYPE__ = "long unsigned int"（64位）
__PTRDIFF_TYPE__ = "long int"
__INT_MAX__ = 2147483647
...
    ↓
预处理时，这些宏可以被头文件使用
    ↓
stddef.h 使用 __SIZE_TYPE__ 定义 size_t
    ↓
代码使用 size_t
```

### C语言头文件之间的包含关系
1. .c文件中只是包含了<stdio.h>,那么数据类型`size_t`在哪里定义的？
`size_t`并不是`stdio.h`中定义的，而是由GCC编译器定义的。`stdio.h`**间接包含**了定义`size_t`的头文件（通常是`stddef.h`或GCC内部头文件）。

Ubuntu(glibc)上，/usr/include/stdio.h 包含了如下代码：
```c
#define __need_size_t
#define __need_NULL
#include <stddef.h>

#define __need___va_list
#include <stdarg.h>
```
关键机制：
- #define __need_size_t 告诉 stddef.h："我只需要 size_t，别给我其他东西"
- #include <stddef.h> 引入 stddef.h
- stddef.h 里用 `typedef __SIZE_TYPE__ size_t; 定义 size_t`

`stddef.h`中定义了很多东西：
```c
#ifndef __SIZE_TYPE__
211 #define __SIZE_TYPE__ long unsigned int
212 #endif
213 #if !(defined (__GNUG__) && defined (size_t))
214 typedef __SIZE_TYPE__ size_t;
215 #ifdef __BEOS__
216 typedef long ssize_t;
217 #endif /* __BEOS__ */
218 #endif /* !(defined (__GNUG__) && defined (size_t)) */
```
`__SIZE_TYPE__` 是 GCC 在编译时根据目标平台（x86_64、ARM等）自动定义的。它的值（如 "long unsigned int"）由 GCC 在配置和编译时根据目标平台决定，定义在 gcc/config/<arch>/<arch>.h 中。

如果 stdio.h 直接 #include <stddef.h>，会把 NULL、offsetof 等全部引入，可能造成：
命名冲突（比如 stdio.h 里也可能定义 NULL）
重复定义（多个头文件都包含 stddef.h）
编译变慢（引入不需要的内容）
所以 glibc 用 __need_size_t 这种选择性包含机制，只取需要的部分。

**验证`size_t`定义**
```bash
# 找到 GCC 内部头文件路径
gcc -print-file-name=include
#输出：/usr/lib/gcc/x86_64-linux-gnu/13/include

# 查看 stddef.h 里 size_t 的定义
grep -n "size_t" $(gcc -print-file-name=include)/stddef.h | head -20
# 输出：如上
```
而 __SIZE_TYPE__ 是编译器内置的宏（前面讨论过）：
```bash
gcc -dM -E - < /dev/null | grep __SIZE_TYPE__
# #define __SIZE_TYPE__ long unsigned int
```

### 检查是否安装了某个特定的包
```bash
dpkg -s libc6-dev libssl-dev | grep Status
```
输出：install ok installed

### 如何查看安装包包含的文件
```bash
dpkg -L libc6-dev
#它会列出libc6-dev包所包含的文件。可以分类列出
# 查看头文件（.h）
dpkg -L libssl-dev | grep "\.h$"

# 查看动态库（.so）
dpkg -L libssl-dev | grep "\.so$"

# 查看静态库（.a）
dpkg -L libssl-dev | grep "\.a$"

# 查看手册页（man）
dpkg -L libssl-dev | grep "man"

# 只看头文件和库
dpkg -L libssl-dev | grep -E "\.(h|so|a)$"
```
### 如何查看某个文件属于哪个包
```bash
dpkg -S /usr/include/openssl/ssl.h
# libssl-dev:amd64: /usr/include/openssl/ssl.h
dpkg -S /usr/lib/x86_64-linux-gnu/libssl.so
#libssl-dev:amd64: /usr/lib/x86_64-linux-gnu/libssl.so
```
### 查看为安装包的文件列表
如果包还没装，可以用 apt-file 查看它包含哪些文件。
```bash
apt install apt-file
apt-file update

# 查看 libssl-dev 包含的所有文件
apt-file list libssl-dev

# 查找某个头文件属于哪个包
apt-file search openssl/ssl.h

# 查找某个库属于哪个包
apt-file search libssl.so
```

### 如何验证开发环境是否完整
以openssl为例，检查三要素：

1. 头文件
ls /usr/include/openssl/ssl.h
或
dpkg -L libssl-dev | grep "openssl/ssl.h"

2. 库文件
ls /usr/lib/x86_64-linux-gnu/libssl.so
ls /usr/lib/x86_64-linux-gnu/libssl.a

3. 手册页
man SSL_new
或
dpkg -L libssl-dev | grep "man3"
