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

## C代码理解
1. volatile 修饰符
```c
#define GPIOA_IDR (*(volatile uint32_t *)0x40010808)
```
这个宏定义是一个指针加一个引用，整体含义：
> 把地址 0x40010808 强制转换为"指向 volatile uint32_t 的指针"，然后解引用它，得到该地址上的 32 位寄存器值。

逐层拆解：
- 第一层: 0x40010808
  一个**整数常量**，表示GPIOA外设IDR(Input Data Register)寄存器的物理地址。这是STM32芯片手册里面规定的硬件地址。
- 第二层: (*(volatile uint32_t *))
  **类型转换(cast)**, 把整数0x40010808解释为“指向volatile uint32_t的指针”。
  其中u32int_t含义为32位无符号整数（寄存器宽度）
  volatile告诉编译器：这个值随时可能被硬件改变，**禁止优化**。
  *表示这是指针类型。
- 第三层: *(...)
   **解引用（dereference）**，把指针0x40010808解引用，得到该地址上的32位寄存器值。

整体含义：
```text
0x40010808          ← 一个地址（整数）
(volatile uint32_t *)0x40010808   ← 把地址变成"指向寄存器的指针"
*(volatile uint32_t *)0x40010808  ← 取出该地址上的值
```
 为什么要volatile:
```c
// 没有 volatile 的危险情况
uint32_t a = *(uint32_t *)0x40010808;
uint32_t b = *(uint32_t *)0x40010808;
// 编译器可能优化为：
// uint32_t a = *(uint32_t *)0x40010808;
// uint32_t b = a;  // ❌ 错误！第二次读取可能值不同
```
加了volatile后：
```c
uint32_t a = *(volatile uint32_t *)0x40010808;
uint32_t b = *(volatile uint32_t *)0x40010808;
// 编译器保证每次都从内存重新读取
```
**原因**：硬件寄存器的值可能在两次读取之间被外设改变（比如按键按下、数据到达）。volatile 禁止编译器做"缓存到寄存器"的优化。

使用示例：
```c
// 定义宏
#define GPIOA_IDR (*(volatile uint32_t *)0x40010808)

// 读取 GPIOA 输入数据寄存器
uint32_t value = GPIOA_IDR;

// 检查某一位（比如第 5 位）
if (GPIOA_IDR & (1 << 5)) {
    // PA5 引脚为高电平
}

// 读取多个位
uint32_t pin_state = GPIOA_IDR & 0xFF;  // 低8位
```
注意：GPIOA_IDR 看起来像变量，但每次使用都会从硬件地址重新读取。

真正的双指针长这样：
```c
uint32_t **pp;  // 指向指针的指针
```

不同的封装方式：
1. 宏定义（最常见，STM32标准库风格）：
```c
#define GPIOA_IDR (*(volatile uint32_t *)0x40010808)
uint32_t v = GPIOA_IDR;
```
2. 指针变量
```c
volatile uint32_t *GPIOA_IDR = (volatile uint32_t *)0x40010808;
uint32_t v = *GPIOA_IDR;
```
3. 结构体映射（最优雅）
```c
typedef struct {
    volatile uint32_t MODER;
    volatile uint32_t OTYPER;
    volatile uint32_t OSPEEDR;
    volatile uint32_t PUPDR;
    volatile uint32_t IDR;
    volatile uint32_t ODR;
    // ...
} GPIO_TypeDef;

#define GPIOA ((GPIO_TypeDef *)0x40010800)
uint32_t v = GPIOA->IDR;
```
这是现代STM32 HAL库的做法，可读性最好。

关键点：
- 中断与主循环共享变量不加volatile：
  会造成编译器优化后，while循环看不到变量更新，程序卡死。
- 滥用volatile：
  如普通局部变量、普通全局变量不要随便加volatile，会降低代码执行效率。
误区：volatile不能保证原子性，多中断并发访问，依然需要关中断保护。

2. static 修饰符
static有两个作用，取决于定义位置：
**函数内部static局部变量**：
生命周期全局，只初始化一次，存放在全局数据区，不是栈。函数退出后，变量保留值。

**函数外部static全局变量**：
变量/函数作用域仅限当前`.c`文件，其他文件无法`extern`引用，主要实现代码隔离，模块化必备。

示例：
```c
void adc_sample(void){
    static uint16_t cnt = 0; // 只初始化一次，函数多次调用，数值持续累加
    cnt++;
}
static uint8_t dev_status; // 仅本文件内可见，外部文件无法访问
static void adc_filter(void) // 私有函数，外部不能调用
{

}
```

**关键点**
- 局部 static 变量不会随函数调用重置，多次调用会保留旧值，容易忽略。
- 多中断同时读写 static 全局变量，存在竞态，需要保护。
- 大量 static 局部变量会占用全局 RAM，不是栈，栈溢出排查时容易忽略。

3. const 修饰符
**const三种写法的含义**：
```c
const int *p;        // 指向 const int 的指针（内容不可改，指针可改）
int *const p;        // const 指针（指针不可改，内容可改）
const int *const p;  // 指向 const 的 const 指针（都不可改）
```
记忆规则：
`const` 在 `*` 左边 → 修饰内容（`*p` 不可改）
`const` 在 `*` 右边 → 修饰指针（`p` 不可改）
示例：
```c
// 错误1：以为 int *const p 是"内容不可改"
int a = 1, b = 2;
int *const p = &a;
p = &b;    // ❌ 编译错误：p 不可改
*p = 10;   // ✅ 可以改内容

// 错误2：以为 const int *p 是"指针不可改"
const int *p = &a;
p = &b;    // ✅ 可以改指针
*p = 10;   // ❌ 编译错误：内容不可改
```

嵌入式场景：
```c
// 寄存器映射常用 const 指针（指针不可变）
#define GPIOA ((GPIO_TypeDef * const)0x40010800)

// 只读数据用 const 内容
const uint8_t lookup_table[256] = { ... };
```

**const变量不能当数组大小**
在 C89/C90 中，const 变量不是编译期常量，不能用于数组大小。
```c
const int SIZE = 10;
int arr[SIZE];  // ❌ C89/C90 错误：SIZE 不是编译期常量
                // ✅ C99 支持 VLA（但嵌入式慎用 VLA）
```

正确做法：
```c
#define SIZE 10          // ✅ 宏，编译期常量
enum { SIZE = 10 };      // ✅ 枚举，编译期常量
int arr[SIZE];

// 或者用静态 const（C++ 可以，C 不行）
```

**const变量被编译器优化掉，无法在调试器观察**
```c
const int debug_flag = 1;

int main(void) {
    if (debug_flag) {  // 编译器可能直接优化为 if (1)
        // ...
    }
}
```
问题：
编译器把 const 变量当作编译期常量，不分配内存
在调试器（GDB/LLDB）中看不到 debug_flag 的值
无法在运行时修改它。

解决办法：
```c
volatile const int debug_flag = 1;  // 加 volatile，强制分配内存
```
嵌入式场景：调试时想用调试器观察某个"常量"，加 volatile。

**const与volatile混用**
嵌入式寄存器既需要 volatile（防优化），也可能需要 const（防写）。
```c
// 只读寄存器：volatile + const
#define GPIOA_IDR (*(volatile const uint32_t *)0x40010808)

// 读写寄存器：只用 volatile
#define GPIOA_ODR (*(volatile uint32_t *)0x4001080C)
```
常见错误：
```c
// ❌ 错误：忘了 volatile，编译器可能优化掉连续读取
#define GPIOA_IDR (*(const uint32_t *)0x40010808)

// ❌ 错误：读写寄存器加了 const，无法写入
#define GPIOA_ODR (*(volatile const uint32_t *)0x4001080C)
GPIOA_ODR = 0xFF;  // ❌ 编译错误
```
记忆：
只读寄存器：volatile const
读写寄存器：volatile

**函数参数中的const陷阱**
- 以为const参数能防止调用者修改
  ```c
  void func(const int *p) {
    // *p = 10;  // ❌ 函数内不能改
  }

  int main(void) {
    int a = 1;
    func(&a);
    a = 10;  // ✅ 调用者仍然可以改 a
  }
  ```
  const只约束函数内部，不约束调用者。

- 返回const指针给调用者，当调用者强制去掉const会导致未定义行为。
```c
const char *get_name(void) {
    return "hello";
}

int main(void) {
    char *p = (char *)get_name();  // ⚠️ 强制去掉 const
    p[0] = 'H';                     // ❌ 未定义行为：修改字符串常量
}
```
字符串常量在只读段，修改会崩溃。

- 指针参数的传递
```c
void func(const int *p);

int main(void) {
    const int a = 10;
    func(&a);   // ✅ OK
    
    int b = 20;
    func(&b);   // ✅ OK（非 const 可以传给 const 参数）
}
```
规则：非const可以传递给const，反之不行。

** const结构体成员的陷阱**
```c
typedef struct {
    const int id;      // ⚠️ const 成员
    int value;
} Item;
```
- 不能整体赋值，当可以初始化时赋值
```c
  Item a = {1, 100};
  Item b = a;  // ✅ 初始化可以
  b = a;       // ❌ 编译错误：const 成员不能被赋值
```
- 不能动态初始化：
  ```c
  Item item;
  item.id = 1; //❌ 编译错误：const 成员必须在初始化时赋值
  ```
- 不能用于数组动态赋值：
  ```c
  Item arr[10];
  arr[0]=1; //❌ 编译错误：
  ```
  嵌入式结构体尽量不要用const成员，该用const指针或封装函数。
**const和#define的选择**
  |特性|const|#define|
  |:---:|:---:|:---:|
  |类型检查|	✅ 有|	❌ 无|
  |调试可见	|✅ 可见	|❌ 不可见|
  |作用域	|✅ 遵循 C 规则|	❌ 文件级|
|编译期常量	|❌ C89 中不是|	✅ 是|
|内存占用	|✅ 可能分配|	❌ 不分配|

**const在中断和多线程中的问题**
```c
const int shared_flag = 0;

void ISR(void) {
    // shared_flag = 1;  // ❌ 编译错误
}

int main(void) {
    while (!shared_flag) {
        // 编译器可能优化为 while (1)，因为 shared_flag 是 const
    }
}
```
问题：
- const 变量编译器可能优化掉读取
- 中断中无法修改
- 即使中断能改，编译器也不知道

正确做法：
```c
volatile int shared_flag = 0;  // ✅ 用 volatile，不用 const

void ISR(void) {
    shared_flag = 1;  // ✅ 可以改
}
```
规则：
- 中断共享变量 → volatile
- 只读常量 → const
- 只读且可能被硬件改变 → volatile const

**const 与 memcpy/memset的冲突**
```c
const char src[] = "hello";
char dst[10];
memcpy(dst, src, 6);  // ✅ OK（src 作为源）

const char dst2[10];
memcpy(dst2, src, 6); // ❌ dst2 是 const，不能作为目的
```
```c
void fill(char *buf, size_t n);  // 需要写入

const char buf[10];
fill(buf, 10);  // ❌ 编译错误
```
常见错误：把 const 数组传给需要写入的函数。

**const函数返回子的问题**
```c
const int get_value(void) {
    return 10;
}
const int x = get_value();  // 和 int x = get_value() 一样
```
const修饰返回值对基本类型毫无意义。只有返回指针时，const 才有意义。
```c
const char *get_name(void) {
    return "hello";  // ✅ 调用者不能修改字符串内容
}
```

**嵌入式对const修饰符的最佳实践**
```c
// 1. 只读寄存器：volatile const
#define GPIOA_IDR (*(volatile const uint32_t *)0x40010808)

// 2. 读写寄存器：volatile
#define GPIOA_ODR (*(volatile uint32_t *)0x4001080C)

// 3. 只读数据表：const
const uint8_t sine_table[256] = { ... };

// 4. 数组大小：宏或枚举
#define BUF_SIZE 128
// 或
enum { BUF_SIZE = 128 };

// 5. 中断共享变量：volatile
volatile int irq_flag = 0;

// 6. 函数参数只读：const 指针
void process(const uint8_t *data, size_t len);

// 7. 调试观察变量：volatile const
volatile const int debug_level = 3;
```
核心原则：
- const 约束的是"谁不能改"，不是"值不会变"
- 硬件寄存器必须 volatile，只读寄存器用 volatile const
- 数组大小用宏/枚举，不用 const
- 中断共享变量用 volatile，不用 const
- 指针 const 位置要分清：左边内容，右边指针

4. 局部变量指针返回会造成悬空指针(dangling pointer):
如下代码：
```c
uint8_t *get_buf(void){
    uint8_t buf[10];
    return buf; // 函数返回后，buf数组会释放，返回的指针会悬空，程序崩溃。
}
```
函数返回后：
- buf 的栈空间被回收（rsp 回退）
- 那块内存变得"可被后续函数调用复用"
- 调用者拿到的指针是悬空指针（dangling pointer）

用`-Wall`编译时，GCC/Clang会提示：
```text
warning: function returns address of local variable [-Wreturn-local-addr]
```
这是必须修复的警告，加上 -Werror 会直接编译失败。
运行时是未定义行为(UB)，同一份代码，不同环境表现完全不同。

**解决方法**
- 方案1:调用者提供缓冲区（推荐）
  ```c
  void get_buf(uint8_t *buf, size_t size) {
    // 填充 buf
    for (size_t i = 0; i < size; i++) {
        buf[i] = i;
    }
   }
  int main(void) {
    uint8_t buf[10];
    get_buf(buf, sizeof(buf));  // ✅ 内存在 main 的栈帧上
    return 0;
  }
```
优点：无内存分配，无泄漏风险，最简单。符合 C 语言"谁分配谁释放"的惯例。

- 方案2:使用static数据
```c
  uint8_t *get_buf(void) {
    static uint8_t buf[10];  // ✅ 静态存储，生命周期贯穿整个程序
    return buf;
  }
```
缺点：只有一份，多次调用共享一块内存，容易造成数据冲突。非线程安全。如果调用者期望独立缓冲区，会出错。
```c
uint8_t *a = get_buf();
uint8_t *b = get_buf();
// a 和 b 指向同一块内存！
```
- 方案3:使用动态分配内存(malloc)
```c
uint8_t *get_buf(void) {
    uint8_t *buf = malloc(10);
    if (buf == NULL) return NULL;
    return buf;
}

int main(void) {
    uint8_t *p = get_buf();
    // 使用 p
    free(p);  // ✅ 调用者负责释放
    return 0;
}
```
优点：每次调用返回独立缓冲区，线程安全。
缺点：调用者必须记得free，否则内存泄露；堆比栈分配慢；需要检查malloc返回值。

- 方案4: 返回结构体（封装数组）
```c
typedef struct {
    uint8_t data[10];
} Buffer;

Buffer get_buf(void) {
    Buffer buf;
    for (int i = 0; i < 10; i++) buf.data[i] = i;
    return buf;  // ✅ 值拷贝，安全
}

int main(void) {
    Buffer b = get_buf();  // 拷贝整个结构体
    return 0;
}
```
优点：值语义，无指针问题。
缺点：大数组拷贝开销大。

5. 寄存器位操作
嵌入式操作寄存器、状态标志、IO配置，几乎全部用位操作。
基础运算符常用模版：
```c
#define BIT_SET(x,n)    ((x) |= (1u << (n)))   // 置1第n位
#define BIT_CLR(x,n)    ((x) &= ~(1u << (n)))  // 清零第n位
#define BIT_READ(x,n)   (((x) >> (n)) & 1u)    // 读取第n位
#define BIT_TOGGLE(x,n) ((x) ^= (1u << (n)))   // 翻转第n位
```
**关键点**：
- 一定要加U无符号，防止移位产生符号扩展 BUG！
- 移位优先级低，不加括号，宏展开优先级错乱，出现意想不到结果。
- 有符号数右移，高位补符号位，不要用 int 做位运算，一律用uint8_t / uint32_t无符号类型。
