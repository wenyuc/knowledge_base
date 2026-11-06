<a id="top"></a>

# 启动代码 _start 与裸机链接

> 归档自 DeepSeek 分享对话：<https://chat.deepseek.com/share/ytrbqq4h7dvcvhajt9>  
> 本主题共 **10 轮**问答（原始全局编号 #067–#078），时间 2026-07-04 ~ 2026-07-16。  
> 所属分组：**ARM 工具链与裸机开发**  
> 提问与回答均为原始内容；回答内小标题层级整体下沉 2 级，`[reference:N]` 占位符已尽量还原为来源链接。  
> [« 返回总索引](./README.md)

## 本主题目录

- [#067 aarch64-elf-ld -nostdlib -nostartfiles -T link.ld -o kernel…](#q067)
- [#068 aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e…](#q068)
- [#069 main.c的代码如下：// main.c - 裸机程序的最小验证示例 // 1. 防止编译器自动插入标准库的启动代码…](#q069)
- [#070 用了方法一，编译有告警aarch64-elf-gcc -mcpu=cortex-a72 -ffreestanding…](#q070)
- [#071 如果将_start放在boot.S中，main.c中不包含_start函数，那么这一行void _start(void)…](#q071)
- [#072 // boot.S .section .text.boot // 定义一个名为 .text.boot 的段 .globl…](#q072)
- [#073 这样就对了吧：nm boot.o | grep _start 0000000000000000 T _start we…](#q073)
- [#076 aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e…](#q076)
- [#077 这样合理一些吗？rm -rf \*.o \*.elf \*.img wenyuc 07-16 11:26 ~/work/ba…](#q077)
- [#078 指令c100 0058 和 ldr x1, 0x80000 是怎么对应起来的？](#q078)

---

<a id="q067"></a>

## #067 · aarch64-elf-ld -nostdlib -nostartfiles -T link.ld -o kernel…

> 🕒 2026-07-04 17:07 ｜ DeepSeek 消息 ID 133–134 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
>aarch64-elf-ld -nostdlib -nostartfiles -T link.ld -o kernel8.elf *.o
aarch64-elf-ld: Error: unable to disambiguate: -nostartfiles (did you mean --nostartfiles ?)
wenyuc 07-04 17:06 ~/work/bare-metal-proj>aarch64-elf-ld -nostdlib --nostartfiles -T link.ld -o kernel8.elf *.o
aarch64-elf-ld: unrecognized option '--nostartfiles'
aarch64-elf-ld: use the --help option for usage information
```

**回答**

这个错误信息非常典型，它揭示了一个关于 `aarch64-elf-ld` 链接器选项的细节差异。问题出在 **`-nostartfiles` 这个选项上**。

##### 🔍 为什么会出现这个错误？

链接器报错的原因是：

1.  `aarch64-elf-ld`（GNU 链接器）**并不直接支持 `-nostartfiles` 这个选项**。
2.  这个选项其实是 **GCC 编译器前端**的选项，而不是 `ld` 链接器本身的选项。

当你在命令行里敲 `aarch64-elf-ld ... -nostartfiles ...` 时，链接器根本认不出 `-nostartfiles` 这个指令，所以它尝试帮你纠正成了 `--nostartfiles`（多了一个 `-`），然而 `ld` 同样也不认 `--nostartfiles`。

##### 🛠️ 解决方案

你的编译和链接流程需要做一些调整。既然 `-nostartfiles` 是 GCC 的选项，你应该**使用 `aarch64-elf-gcc` 作为链接器前端**，而不是直接调用 `ld`。

**推荐方案：改用 `aarch64-elf-gcc` 进行链接**

```bash
# 使用 aarch64-elf-gcc 来执行链接（它会正确解析 -nostartfiles 并调用 ld）
aarch64-elf-gcc -nostdlib -nostartfiles -T link.ld -Wl,-e,_start -o kernel8.elf *.o
```

**这里的关键参数是：**
- **`-nostdlib`**：GCC 选项，告诉编译器前端不要自动链接标准库（已被正确识别）。
- **`-nostartfiles`**：GCC 选项，告诉编译器前端不要链接默认的启动文件（如 `crt0.o`），现在也能正常工作了。
- **`-Wl,-e,_start`**：这是一个 `-Wl` 选项，用于将逗号后的参数（`-e,_start`）直接传递给底层的链接器 `ld`。`-e _start` 会显式地设置程序的入口点为 `_start`，这是启动代码的通常入口。

##### 💡 如果坚持使用 `aarch64-elf-ld` 该怎么办？

如果你确实想直接使用 `aarch64-elf-ld`，你需要**手动指定入口点**，并**避免使用不被支持的选项**。毕竟裸机程序没有默认的启动文件，这个选项本身就不是必需的：

```bash
# 直接使用 ld：不需要 -nostartfiles，用 -e 指定入口即可
aarch64-elf-ld -nostdlib -T link.ld -e _start -o kernel8.elf *.o
```

##### 💎 总结
核心区别在于：
*   **`-nostartfiles` 是 GCC 编译器的选项**，`ld` 链接器无法识别。
*   最稳定的做法是**使用 `aarch64-elf-gcc` 作为链接器前端**，并利用 `-Wl,` 选项将入口点参数传递给 `ld`。这是许多嵌入式项目中管理链接的通用方法。

简单来说，为了解决这个问题，你可以直接使用 `aarch64-elf-gcc` 来完成链接步骤：`aarch64-elf-gcc -nostdlib -nostartfiles -T link.ld -Wl,-e,_start -o kernel8.elf *.o`。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q068"></a>

## #068 · aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e…

> 🕒 2026-07-04 19:20 ｜ DeepSeek 消息 ID 135–136 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
>aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e,_start -o kernel8.elf main.o
/usr/local/opt/aarch64-elf-binutils/bin/aarch64-elf-ld: warning: cannot find entry symbol _start; defaulting to 0000000000080000这个报错是否是因为linker.ld文件中没有定义start？
```

**回答**

**是的，你的直觉完全正确！** 这个警告的核心原因，就是链接器找不到名为 `_start` 的入口符号（entry symbol）。不过，这不一定是因为 `linker.ld` 文件里“没有定义”，而是在你提供的 `main.o` 文件中**缺少一个名为 `_start` 的函数实现**。

##### 🔍 错误信息解析

这条警告信息非常关键，它告诉我们两件事：

1.  **`warning: cannot find entry symbol _start`**：链接器（`ld`）在所有的输入文件（`main.o` 和库文件中）进行符号查找，试图找到一个名为 `_start` 的全局符号，但并没有找到。
2.  **`defaulting to 0000000000080000`**：因为找不到，链接器就使用了一个默认值。这个 `0x80000` 恰好是你在 `linker.ld` 中通过 `. = 0x80000;` 设置的起始地址。

##### 📝 深入理解：入口点（Entry Point）

这里涉及到一个重要的概念：**入口点（Entry Point）**。

*   **入口点是什么**：对于操作系统或裸机程序来说，入口点就是 CPU 在上电或复位后，第一条要执行的指令的地址。
*   **`-e _start` 的作用**：你在命令行中指定了 `-Wl,-e,_start`，这相当于明确告诉链接器：“请把我的程序的入口点符号名设置为 `_start`。”
*   **符号解析**：为了满足你的这个要求，链接器会在所有目标文件（`.o` 文件）中查找一个名为 `_start` 的**全局符号**。这个符号通常指向一段汇编代码，负责最底层的 CPU 初始化。

##### 🚀 如何解决？

解决这个问题的方法也很直接：你需要为你的程序提供一个名为 `_start` 的入口点。通常，这是一段用汇编语言编写的启动代码。

在你的项目目录下创建一个名为 `boot.S` 的文件（`S` 是大写，表示这是预处理器需要处理的汇编文件）：

```assembly
// boot.S
.section .text.boot      // 定义一个名为 .text.boot 的段
.globl _start            // 声明 _start 为全局符号

_start:
    // 1. 设置栈指针 (SP)
    // 这里用 0x80000 作为临时栈顶，真正项目中需要移到更高地址
    ldr x1, =0x80000
    mov sp, x1

    // 2. 跳转到 main 函数
    // 注意：需要先确保 main.c 声明了 main 函数
    bl main

    // 3. 如果 main 意外返回，则死循环
halt:
    wfe
    b halt
```

##### 🔧 更新你的构建流程

创建好 `boot.S` 后，你需要将它也纳入编译流程中：

```bash
# 1. 编译汇编启动文件
aarch64-elf-gcc -c boot.S -o boot.o

# 2. 编译你已有的 main.c
aarch64-elf-gcc -c main.c -o main.o

# 3. 链接所有目标文件
aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e,_start -o kernel8.elf main.o boot.o
```

现在，链接器就能在 `boot.o` 文件中找到 `_start` 这个符号了，警告信息自然就会消失。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q069"></a>

## #069 · main.c的代码如下：// main.c - 裸机程序的最小验证示例 // 1. 防止编译器自动插入标准库的启动代码…

> 🕒 2026-07-04 19:23 ｜ DeepSeek 消息 ID 137–138 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
main.c的代码如下：// main.c - 裸机程序的最小验证示例

// 1. 防止编译器自动插入标准库的启动代码
void _start(void) __attribute__((naked, section(".text._start")));

// 2. 主程序入口，模拟一个死循环
void _start(void) {
    // 这里什么也不做，只是一个无限循环
    // 相当于裸机程序的 "停在这里"
    while(1);
}

// 3. 占位函数，防止链接时因缺少标准库函数而报错
// 某些 GCC 版本会隐式生成对 memset 等函数的调用
void __attribute__((weak)) _init(void) {}
void __attribute__((weak)) _fini(void) {}这里是有 _start()函数，为什么link的时候找不到呢？
```

**回答**

这是一个非常经典且容易被忽略的细节问题！你的 `main.c` 中确实定义了 `_start` 函数，**但它并没有被编译成一个可供链接器识别的 `_start` 符号**。

问题的关键就在于这一行声明：

```c
void _start(void) __attribute__((naked, section(".text._start")));
```

##### 🔍 问题根源

`section(".text._start")` 这个属性，明确地告诉编译器：“请将这个 `_start` 函数放在名为 **`.text._start`** 的段中，而不是默认的 `.text` 段。”

因此，虽然你的函数名叫 `_start`，但它在目标文件（`main.o`）中的最终归宿是 `.text._start` 这个段。

链接器在查找入口符号 `_start` 时，是在全局符号表中搜索名为 `_start` 的符号。符号的名字与段的名字是两个完全不同的概念，链接器并**不会**因为某个段的名字叫 `.text._start`，就自动将它里面的内容识别为入口点。

也就是说，你的 `_start` 函数虽然存在，但它的“名字”并没有被导出为一个可供链接器识别的入口符号。

##### 🛠️ 解决方法

有两个思路可以修正这个问题：

**方法一：移除 `section` 属性（最简单）**

如果你不介意 `_start` 函数被放在 `.text` 段的默认位置，可以直接移除 `section(".text._start")` 属性：

```c
void _start(void) __attribute__((naked));
```

这样编译器就会把 `_start` 函数放在默认的 `.text` 段中，链接器就能顺利找到它了。

**方法二：保留 `section` 属性，显式指定入口**

如果出于某种原因，你希望 `_start` 函数保留在 `.text._start` 段中（比如配合 `KEEP(*(.text.boot))` 使用），你可以通过两种方式来指定入口点：

1.  **在链接脚本 `linker.ld` 中指定**：
    你可以在链接脚本的开头添加 `ENTRY` 指令，告诉链接器入口点符号的名称，而不必依赖于段名。
    ```ld
    ENTRY(_start)
    ```

2.  **在链接命令中指定**：
    你已经在使用 `-Wl,-e,_start`，但这个方法依赖于 `_start` 这个符号存在于全局符号表中。

##### 💡 总结建议

你的 `main.c` 定义了 `_start` 函数，但链接器找不到它的根本原因，是 **`section(".text._start")` 属性改变了该函数在目标文件中的“存放位置”（段名），却没有导出 `_start` 这个入口符号**。

对于初学者来说，建议采用**方法一**，移除 `section` 属性，让 `_start` 函数回归到默认的 `.text` 段，这样能避免很多潜在的混淆。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q070"></a>

## #070 · 用了方法一，编译有告警aarch64-elf-gcc -mcpu=cortex-a72 -ffreestanding…

> 🕒 2026-07-04 19:30 ｜ DeepSeek 消息 ID 139–140 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
用了方法一，编译有告警>aarch64-elf-gcc -mcpu=cortex-a72 -ffreestanding -nostdlib -c main.c -o main.o
main.c:4:1: warning: 'naked' attribute directive ignored [-Wattributes]
    4 | void _start(void) __attribute__((naked));
      | ^~~~
```

**回答**

这个警告说明 `naked` 属性在你的代码中**被编译器忽略了**。这是一个非常好的发现，也是初学者在使用 `naked` 属性时常遇到的问题。

##### 🔍 为什么会忽略 `naked`？

`naked` 属性的作用是告诉编译器："**这个函数不要生成任何栈帧（prologue/epilogue）代码，函数体就是我写的纯汇编**"。

但在你的代码中：
```c
void _start(void) __attribute__((naked));

void _start(void) {
    while(1);
}
```

你写的是一个 **C 语言的 while(1) 循环**，而不是纯汇编代码。编译器在编译时发现：你声称这是一个 `naked` 函数，但函数体里却是 C 语句，它会觉得"这有矛盾"，于是**忽略 `naked` 属性**，按普通 C 函数编译。

##### 💡 解决方案

对于裸机程序的 `_start`，你有两个选择：

###### 方法一：在汇编文件中实现 `_start`（推荐）

这是最干净、最可靠的方式。将 `_start` 用汇编语言来写，逻辑完全可控：

**`boot.S`（汇编启动文件）**：
```assembly
.section .text.boot
.globl _start

_start:
    // 1. 设置栈指针（临时）
    ldr x1, =0x80000
    mov sp, x1

    // 2. 跳转到 C 语言的 main 函数
    bl main

    // 3. main 返回后死循环
halt:
    wfe
    b halt
```

**`main.c`（C 语言主程序）**：
```c
// 只有主逻辑，不再定义 _start
int main(void) {
    while(1);
}
```

**编译命令**：
```bash
# 编译汇编文件
aarch64-elf-gcc -c boot.S -o boot.o

# 编译 C 文件
aarch64-elf-gcc -mcpu=cortex-a72 -ffreestanding -nostdlib -c main.c -o main.o

# 链接（将 _start 作为入口）
aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e,_start -o kernel8.elf main.o boot.o
```

###### 方法二：移除 `naked` 属性（快速但不推荐）

如果你只是想快速验证，可以完全移除 `naked` 属性，让编译器将 `_start` 当作普通 C 函数处理：

```c
void _start(void) {
    while(1);
}
```

**但不推荐**，因为编译器会插入标准函数序言和尾声，可能会干扰 `_start` 作为程序入口的行为。

##### 💎 总结

| 方案 | 优点 | 缺点 |
| :--- | :--- | :--- |
| **汇编 `boot.S`** | 最可靠、完全可控、是最佳实践 | 需要了解一点 ARM64 汇编语法 |
| **纯 C 实现 `_start`** | 快速方便，适合验证 | 可能产生不可预期的编译器行为 |

对于裸机开发，**将 `_start` 放在汇编文件中**是一个标准做法。这意味着你的 `main.c` 中不再包含 `_start` 函数，你的链接脚本 `linker.ld` 也无需 `ENTRY(_start)`，因为 `-Wl,-e,_start` 已经指定了入口点，而 `_start` 符号只在 `boot.o` 中存在。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q071"></a>

## #071 · 如果将_start放在boot.S中，main.c中不包含_start函数，那么这一行void _start(void)…

> 🕒 2026-07-16 10:49 ｜ DeepSeek 消息 ID 141–142 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
如果将_start放在boot.S中，main.c中不包含_start函数，那么这一行void _start(void) __attribute__((naked));可以不要了吧？
```

**回答**

**是的，完全可以，而且应该删除！**

一旦你决定将 `_start` 的实现移到 `boot.S` 中，`main.c` 就不再需要包含 `_start` 函数及其声明。以下是更详细的原因和优化建议：

##### 🧹 `main.c` 的清理原则

在裸机开发中，`main.c` 应该只包含**C语言层面的逻辑**，而与硬件启动、栈指针设置等相关的汇编代码，则统一放在 `boot.S` 中。这是一种将“硬件入口”与“应用逻辑”清晰分离的设计方式。

**你原来的 `main.c` 可以简化为**：
```c
// main.c
int main(void) {
    while(1);
}
```

它不再包含以下内容：
*   ❌ `void _start(void) __attribute__((naked));`（这是多余的）
*   ❌ `_init` 和 `_fini` 占位函数（除非链接时报错，才需要保留）

##### ✅ 如何正确验证？

你可以通过以下步骤来检查符号的归属关系，确保所有符号都位于正确的位置：

1.  **检查 `_start` 在哪个文件中**：
    ```bash
    nm boot.o | grep _start
    ```
    应该看到 `0000000000000000 T _start`，说明 `_start` 是 `boot.o` 中定义的全局符号。

2.  **检查 `main` 在哪个文件中**：
    ```bash
    nm main.o | grep main
    ```
    应该看到 `main` 符号，说明 `main` 函数定义在 `main.o` 中。

3.  **检查链接后 `_start` 是否为入口**：
    ```bash
    aarch64-elf-readelf -h kernel8.elf | grep Entry
    ```
    应该显示入口地址为 `0x80000` 或你设置的地址。

##### 💡 保持链接器脚本的一致性

你还需要确保 `linker.ld` 中 `.text.boot` 段（即 `boot.S` 所在的段）被放置在内核镜像的**最开头**。这是通过 `KEEP(*(.text.boot))` 和 `*(.text*)` 的顺序来保证的。

---

**总结**：删除 `main.c` 中的 `_start` 声明是正确的，因为该函数的实现（即入口点）已经由 `boot.S` 提供，`main.c` 不需要也不应该再声明它。这样可以让代码职责更清晰，也减少了潜在的混淆。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q072"></a>

## #072 · // boot.S .section .text.boot // 定义一个名为 .text.boot 的段 .globl…

> 🕒 2026-07-16 10:58 ｜ DeepSeek 消息 ID 143–144 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
// boot.S
.section .text.boot      // 定义一个名为 .text.boot 的段
.globl _start            // 声明 _start 为全局符号

_start:
    // 1. 设置栈指针 (SP)
    // 这里用 0x80000 作为临时栈顶，真正项目中需要移到更高地址
    ldr x1, =0x80000
    mov sp, x1

    // 2. 跳转到 main 函数
    // 注意：需要先确保 main.c 声明了 main 函数
    bl main

    // 3. 如果 main 意外返回，则死循环
halt:
    wfe
    b halt 然后用>aarch64-elf-gcc -c boot.S -o boot.o，但是nm boot.o | grep _start没有返回0000000000000000 T _start
```

**回答**

你这个现象非常典型！`nm boot.o | grep _start` 没有输出，说明 `_start` 符号**没有被正确导出为全局符号**。这通常是汇编语法细节问题导致的。

##### 🔍 可能的原因和解决方案

###### 1. 检查 `boot.S` 中是否有不可见字符

最常见的原因是文件开头的注释或格式问题。请确认你的 `boot.S` **第一行就是** `.section .text.boot`，没有任何前导字符（包括空格、BOM 头等）。

**错误的写法**（可能被注释干扰）：
```assembly
// boot.S          ← 如果这行注释在文件最开头，可能干扰汇编器
.section .text.boot
.globl _start
```

**正确的写法**：
```assembly
.section .text.boot
.globl _start

_start:
    ldr x1, =0x80000
    mov sp, x1
    bl main
halt:
    wfe
    b halt
```

###### 2. 检查你查看的是正确的文件

确保 `nm` 查看的是你刚刚编译生成的 `boot.o`：

```bash
# 先确认文件存在
ls -la boot.o

# 查看所有符号（不 grep）
nm boot.o
```

如果 `nm boot.o` 输出为空或只有很少的符号，说明编译过程可能有问题。

###### 3. 尝试使用 `aarch64-elf-objdump` 确认

有时候 `nm` 的输出格式可能不同，可以换一个工具验证：

```bash
aarch64-elf-objdump -t boot.o | grep _start
```

###### 4. 检查汇编语法是否正确

你的 `boot.S` 中的 `ldr x1, =0x80000` 这种伪指令写法是 GNU 汇编器支持的。如果汇编器不支持这种写法（某些旧版本），可能会生成错误的符号表。

可以改用更保守的写法验证：
```assembly
.section .text.boot
.globl _start

_start:
    mov x1, #0x80000
    mov sp, x1
    bl main
halt:
    wfe
    b halt
```

##### 🧪 快速验证方法

你可以用以下命令直接查看汇编器生成的符号表，确认 `_start` 是否被正确识别：

```bash
# 1. 先查看汇编后的符号
aarch64-elf-objdump -t boot.o

# 2. 如果还是没有，检查目标文件格式
file boot.o
```

##### 💡 如果问题仍然存在

如果以上方法都无法解决，请提供以下信息以便进一步诊断：

1. **`nm boot.o` 的完整输出**（不 grep）
2. **`cat -A boot.S` 的输出**（检查是否有不可见字符）
3. **`aarch64-elf-gcc --version` 的输出**

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q073"></a>

## #073 · 这样就对了吧：nm boot.o | grep _start 0000000000000000 T _start we…

> 🕒 2026-07-16 11:05 ｜ DeepSeek 消息 ID 145–146 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
这样就对了吧：>nm boot.o | grep _start
0000000000000000 T _start
wenyuc 07-16 10:59 ~/work/bare-metal-proj>nm main.o | grep main
0000000000000000 T main
wenyuc 07-16 11:04 ~/work/bare-metal-proj>aarch64-elf-readelf -h kernel8.elf | grep Entry
  Entry point address:               0x80000
```

**回答**

**完全正确！** 你的验证结果非常完美，这说明整个工具链和代码都已经正确配置，准备工作全部就绪。这里为你梳理一下这些关键输出所证明的事实：

1.  **`nm boot.o | grep _start` 输出 `0000000000000000 T _start`**：
    *   证明 `_start` 符号已经成功定义为**全局（T）**符号，并且它在 `boot.o` 目标文件中的偏移是 `0`，这是正确的起始位置。

2.  **`nm main.o | grep main` 输出 `0000000000000000 T main`**：
    *   证明 `main` 函数也已在 `main.o` 中正确定义为全局符号，可以被 `_start` 中的 `bl main` 指令正确调用。

3.  **`aarch64-elf-readelf -h kernel8.elf | grep Entry` 输出 `Entry point address: 0x80000`**：
    *   这是最关键的一步！它证明链接器已经根据你的命令行选项 `-Wl,-e,_start` 和链接脚本（`linker.ld`）中的 `. = 0x80000;` 设置，成功将 `_start` 符号解析为入口点，并将其地址设置为 `0x80000`。

##### 🎯 总结

这三条命令的输出，从**符号定义**、**函数实现**到**最终入口地址**，形成了一条完整的证据链，准确无误地证明你的开发环境、代码、编译和链接流程都已正确设置。

**你现在已经成功构建出了一个能够被树莓派加载的 `kernel8.elf` 文件了**。下一步，你可以使用 `aarch64-elf-objcopy -O binary kernel8.elf kernel8.img` 命令将其转换为原始的二进制镜像文件，并放入 SD 卡的启动分区中，在真实的树莓派硬件上验证它的运行。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q076"></a>

## #076 · aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e…

> 🕒 2026-07-16 11:27 ｜ DeepSeek 消息 ID 151–152 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
>aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e,_start -o kernel8.elf boot.o main.o

/usr/local/opt/aarch64-elf-binutils/bin/aarch64-elf-ld: warning: kernel8.elf has a LOAD segment with RWX permissions
```

**回答**

这个 `RWX` 警告是一个**重要的安全提醒**，但对你当前的学习阶段来说，它**不是致命错误，可以暂时忽略**。

##### 🔍 警告含义

`LOAD segment with RWX permissions` 意味着你的内核镜像中存在一个内存段，它同时被赋予了**读（R）、写（W）和执行（X）**三种权限。在现代操作系统中，这是一种不安全的配置，因为它可能为缓冲区溢出等攻击提供可执行的恶意代码空间。

##### 🧐 为什么会出现？

在标准的 `linker.ld` 脚本中，通常会将所有段（`.text`、`.data`、`.bss` 等）放在一个连续的区域内。链接器为了简化，可能会将整个区域标记为 `RWX`，而不是为代码（`RX`）、数据（`RW`）等分别设置权限。

这在你使用 `-nostdlib` 和自定义链接脚本时非常常见，因为操作系统（或加载器）通常会在加载时根据程序头（Program Header）来设置内存页权限。

##### ✅ 如何解决？（可选）

如果你希望在生成 `kernel8.elf` 时消除这个警告，可以在链接脚本中添加 `PHDRS` 命令，显式地划分内存段并指定权限。

在 `linker.ld` 的开头添加以下内容：

```ld
PHDRS
{
    text PT_LOAD FLAGS(5);  /* 5 = 读 (4) + 执行 (1) */
    data PT_LOAD FLAGS(6);  /* 6 = 读 (4) + 写 (2) */
}
```

然后，在每个输出段（`SECTIONS` 内）使用 `:` 指定它属于哪个程序头：

```ld
SECTIONS
{
    . = 0x80000;

    .text : {
        KEEP(*(.text.boot))
        *(.text .text.*)
    } :text

    .rodata : {
        *(.rodata .rodata.*)
    } :data

    .data : {
        *(.data .data.*)
    } :data

    .bss (NOLOAD) : {
        . = ALIGN(16);
        __bss_start = .;
        *(.bss .bss.*)
        *(COMMON)
        __bss_end = .;
    } :data
}
```

这样，生成的 ELF 文件就会包含明确的 `LOAD` 段权限信息，`RWX` 警告就不会再出现了。

##### 💡 学习建议

对于入门阶段的学习，**可以先忽略这个警告**。因为它主要影响的是 ELF 文件格式的元数据，而实际上你最终会通过 `objcopy -O binary` 将其转换为原始二进制文件（`.img`），此时段权限信息会被完全剥离，只保留纯粹的机器码。只要你能正常生成 `kernel8.img` 并成功启动，这个警告就无关紧要。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q077"></a>

## #077 · 这样合理一些吗？rm -rf \*.o \*.elf \*.img wenyuc 07-16 11:26 ~/work/ba…

> 🕒 2026-07-16 11:31 ｜ DeepSeek 消息 ID 153–154 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
这样合理一些吗？>rm -rf *.o *.elf *.img
wenyuc 07-16 11:26 ~/work/bare-metal-proj>ls -l
total 24
-rw-r--r--@ 1 wenyuc  staff   466 16 Jul 10:59 boot.S
-rw-r--r--@ 1 wenyuc  staff  1093  3 Jul 21:55 linker.ld
-rw-r--r--@ 1 wenyuc  staff   185 16 Jul 10:51 main.c
-rw-r--r--@ 1 wenyuc  staff     0 16 Jul 11:00 note.txt
wenyuc 07-16 11:26 ~/work/bare-metal-proj>aarch64-elf-gcc -c boot.S -o boot.o
wenyuc 07-16 11:26 ~/work/bare-metal-proj>aarch64-elf-gcc -mcpu=cortex-a72 -ffreestanding -nostdlib -c main.c -o main.o
wenyuc 07-16 11:26 ~/work/bare-metal-proj>aarch64-elf-gcc -nostdlib -nostartfiles -T linker.ld -Wl,-e,_start -o kernel8.elf boot.o main.o

/usr/local/opt/aarch64-elf-binutils/bin/aarch64-elf-ld: warning: kernel8.elf has a LOAD segment with RWX permissions
wenyuc 07-16 11:29 ~/work/bare-metal-proj>aarch64-elf-objcopy -O binary kernel8.elf kernel8.img
wenyuc 07-16 11:29 ~/work/bare-metal-proj>ls -l kernel8.img
-rwxr-xr-x@ 1 wenyuc  staff  36 16 Jul 11:29 kernel8.img
wenyuc 07-16 11:29 ~/work/bare-metal-proj>xxd kernel8.img
00000000: c100 0058 3f00 0091 0600 0094 5f20 03d5  ...X?......._ ..
00000010: ffff ff17 0000 0000 0000 0800 0000 0000  ................
00000020: 0000 0014                                ....
wenyuc 07-16 11:29 ~/work/bare-metal-proj>
```

**回答**

**非常合理！而且已经取得了突破性的进展！**

你现在的输出结果非常正常，`kernel8.img` 从之前的 4 字节变成了 **36 字节**，这是一个很好的信号，说明 `main.o` 和 `boot.o` 现在已经被正确链接进去了。

##### 🔍 逐行分析 `xxd kernel8.img` 的输出

让我们来看看这 36 字节都包含了什么（按 4 字节一组，即一条 ARM64 指令）：

```text
c100 0058      // ldr x1, 0x80000  (加载地址)
3f00 0091      // mov sp, x1       (设置栈指针)
0600 0094      // bl 0x18          (跳转到 main，偏移量 0x18)
5f20 03d5      // wfe              (等待事件)
ffff ff17      // b 0xfffffffc     (回跳，形成死循环)
0000 0000      // (数据：占位符)
0000 0800      // (数据：0x80000，即 ldr 指令加载的地址)
0000 0000      // (数据：对齐填充)
0000 0014      // (数据：?)
```

##### 💡 各个部分解读

1.  **`bl 0x18` 已经生效**：`main` 函数现在确实被链接到镜像中，`bl` 指令会跳转到 `0x18` 偏移处执行。

2.  **偏移 `0x1C`（即第 28 字节）处**：存放着数据 `0x0000_0000_0000_0800`，这是 `ldr x1, =0x80000` 的“文字池”（literal pool）。这条伪指令在编译时会将常量 `0x80000` 存放在这里，运行时由 `ldr` 指令加载到寄存器 `x1` 中。

3.  **末尾的 `0000 0014`**：这很可能是 `main` 函数的返回值。由于你的 `main` 函数是 `int main(void)`，`aarch64-elf-gcc` 默认会生成 `mov w0, wzr`（将 `w0` 置零）作为返回语句的末尾。

##### 🎯 结论

**一切正常！** 你已经成功地将 `_start` 和 `main` 链接起来，并生成了一个功能完整的、可引导的 ELF 文件及对应的二进制镜像。

`kernel8.img` 的大小为 36 字节是合理的，对于裸机程序来说，这已经是向前迈出了非常扎实的一步了。下一步就是将它烧录到 SD 卡并在树莓派 4 上测试运行。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q078"></a>

## #078 · 指令c100 0058 和 ldr x1, 0x80000 是怎么对应起来的？

> 🕒 2026-07-16 12:27 ｜ DeepSeek 消息 ID 155–156 ｜ 主题：启动代码 _start 与裸机链接

**提问**

```text
指令c100 0058 和 ldr x1, 0x80000  是怎么对应起来的？
```

**回答**

`c100 0058` 和 `ldr x1, 0x80000` 的对应关系，涉及到 ARM64 指令编码和汇编器（Assembler）的一个重要机制——**文字池（Literal Pool）**。

你的 `boot.S` 中写的是：
```assembly
ldr x1, =0x80000
```

这是一个 **伪指令（Pseudo-instruction）**，汇编器不会把它翻译成一条对应的机器码，而是会根据情况将其拆分成不同的指令。`c100 0058` 正是这个拆分后的结果。

##### 🔍 完整翻译过程

当汇编器看到 `ldr x1, =0x80000` 时，它会做以下处理：

1.  **在代码段末尾或附近创建一个“文字池”**：它会把 `0x80000` 这个立即数存入这个池子中，并在内存中为其分配位置。
2.  **替换为一条 PC 相对寻址的加载指令**：使用 `ldr x1, [PC, #offset]`，其中 `offset` 是当前指令地址到文字池中 `0x80000` 位置的偏移量。

你看到的 `0000 0800` 位于 `kernel8.img` 偏移 `0x20` 处，它就是文字池中的 `0x80000`。`c100 0058` 中的 `c1` 是操作码，`0000 0058` 则是编码后的偏移量部分，它告诉 CPU 从当前 PC 加 `0x18` 字节（即 `0x20` 处）读取数据。

##### 🛠️ 验证过程

你可以用反汇编命令来验证这个对应关系：

```bash
# 反汇编 boot.o 目标文件
aarch64-elf-objdump -d boot.o

# 你应该会看到类似输出：
# 0000000000000000 <_start>:
#    0:   c1000058    ldr x1, 0x0        // 这里显示为 0x0，偏移在链接后才确定
#    4:   3f000091    mov sp, x1
#    8:   06000094    bl 0x0             // 这里的 0x0 是占位，链接后填充

# 链接后反汇编 kernel8.elf 会显示最终地址：
aarch64-elf-objdump -d kernel8.elf
```

##### 💡 为什么不用 `mov` 直接加载？

ARM64 的 `mov` 指令只能加载 16 位的立即数，而 `0x80000` 超出了这个范围。因此使用 `ldr` 从文字池加载是一种标准解决方案。此外，使用 `ldr x1, =0x80000` 的好处在于，如果 `0x80000` 能够被 `movz` 或 `movk` 指令组合加载，汇编器可能会进行这种优化；否则，它就会退回到文字池的方式。你当前的输出显示它采用了文字池的路径，这是一种正常的汇编器行为。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---
