<a id="top"></a>

# 链接脚本 linker.ld 详解

> 归档自 DeepSeek 分享对话：<https://chat.deepseek.com/share/ytrbqq4h7dvcvhajt9>  
> 本主题共 **8 轮**问答（原始全局编号 #058–#066），时间 2026-07-03 ~ 2026-07-03。  
> 所属分组：**ARM 工具链与裸机开发**  
> 提问与回答均为原始内容；回答内小标题层级整体下沉 2 级，`[reference:N]` 占位符已尽量还原为来源链接。  
> [« 返回总索引](./README.md)

## 本主题目录

- [#058 如何通过链接脚本将树莓派4（64位）的裸机目标文件.o文件链接成树莓派能识别的格式呢？](#q058)
- [#060 这个linker.ld文件是手敲的吗？能否通过工具生成？](#q060)
- [#061 有没有教程讲linker.ld中的sections](#q061)
- [#062 linker.ld中这一段怎么理解“.text : { KEEP(\*(.text.boot)) \*(.text .tex…](#q062)
- [#063 准确的说，是不是将从所有输入文件（.o 文件）中收集来的 .text、.text.\* 和 .gnu.linkonce.t…](#q063)
- [#064 这一段：“ .rodata : { \*(.rodata .rodata.\* .gnu.linkonce.r\*) }”意思…](#q064)
- [#065 这一段“.bss (NOLOAD) : { . = ALIGN(16); __bss_start = .; \*(.bss…](#q065)
- [#066 \*(COMMON)是什么意思？](#q066)

---

<a id="q058"></a>

## #058 · 如何通过链接脚本将树莓派4（64位）的裸机目标文件.o文件链接成树莓派能识别的格式呢？

> 🕒 2026-07-03 13:51 ｜ DeepSeek 消息 ID 115–116 ｜ 主题：链接脚本 linker.ld 详解

**提问**

如何通过链接脚本将树莓派4（64位）的裸机目标文件.o文件链接成树莓派能识别的格式呢？

**回答**

将 `.o` 文件链接成树莓派4（64位）能识别的格式，核心是两步：**编写一个正确的链接脚本（linker script）**，然后**使用链接器（ld）和格式化工具（objcopy）生成最终的 `.img` 镜像文件**。

这个过程主要是告诉链接器如何将你的代码和数据，按照树莓派的启动要求，精确地放置在内存中。

##### 🗺️ 关键一步：编写链接脚本（linker.ld）

链接脚本是这个过程的核心，它指导链接器如何将各个目标文件中的段（section）组合起来，并指定它们在内存中的最终地址[citation:1][citation:4]。

对于树莓派4的64位启动，你需要创建一个`linker.ld`文件。以下是一个基于常见教程的基础模板和说明[citation:1][citation:8]：

```ld
SECTIONS {
    /* 1. 设定起始地址为 0x80000 */
    /* 这是树莓派GPU固件加载64位内核镜像的默认地址 */
    . = 0x80000;

    /* 2. 最先放置 .text.boot 段 */
    /* 这确保了你的启动汇编代码被放在镜像的最开头 */
    .text : {
        KEEP(*(.text.boot))
        *(.text .text.* .gnu.linkonce.t*)
    }

    /* 3. 放置只读数据 */
    .rodata : {
        *(.rodata .rodata.* .gnu.linkonce.r*)
    }

    /* 4. 放置已初始化的数据 */
    .data : {
        *(.data .data.* .gnu.linkonce.d*)
    }

    /* 5. 放置 .bss 段（未初始化的全局/静态变量） */
    /* 这个段需要在启动时被清零，所以记录其起始和结束地址 */
    .bss (NOLOAD) : {
        . = ALIGN(16);
        __bss_start = .;
        *(.bss .bss.*)
        *(COMMON)
        __bss_end = .;
    }

    /* 6. 可选：丢弃一些无用的标准段，以减小文件大小 */
    /DISCARD/ : { *(.comment) *(.gnu*) *(.note*) *(.eh_frame*) }
}

/* 导出BSS段的大小，供启动代码使用 */
__bss_size = (__bss_end - __bss_start) >> 3;
```

这个脚本主要做了几件事：定义了整个程序的起始地址为 `0x80000`[citation:1][citation:8][citation:11]，指定了各段（`.text`、`.data`、`.bss`）的内存布局[citation:1][citation:4]，并导出了BSS段的起始、结束地址和大小，供汇编启动代码在C语言 `main` 函数运行前将其清零[citation:8]。

##### 🔧 第二步：链接与镜像生成

有了链接脚本，就可以通过两步命令生成树莓派能直接加载的镜像文件。

1.  **链接（Linking）**：使用 `aarch64-elf-ld` 链接器，将所有 `.o` 文件按照 `linker.ld` 的规则组合成一个ELF格式的可执行文件（`.elf`）[citation:8]。

    ```bash
    aarch64-elf-ld -nostdlib -nostartfiles -T link.ld -o kernel8.elf *.o
    ```

2.  **格式转换（Objcopy）**：树莓派加载的是纯二进制镜像，不包含ELF头信息。用 `aarch64-elf-objcopy` 将ELF文件转换为原始的二进制文件（`.img`）[citation:1][citation:4][citation:8]。

    ```bash
    aarch64-elf-objcopy -O binary kernel8.elf kernel8.img
    ```

##### 📝 文件命名与放置

最后一步是命名和放置生成的镜像文件，以便树莓派能够识别和加载：

*   **文件命名**：对于树莓派4的64位启动，**必须**将最终的二进制文件命名为 **`kernel8.img`**[citation:8][citation:11]。这个特定的命名会让GPU固件以AArch64（64位）模式引导你的内核[citation:8]。
*   **放置位置**：将 `kernel8.img` 复制到SD卡**第一个分区（FAT32格式的/boot分区）**。
*   **配置文件**：确保SD卡`/boot/config.txt`中至少包含 `arm_64bit=1` 这一行，以强制启用64位模式[citation:8]。

完成这一步后，将SD卡插入树莓派并上电，GPU固件就会将你的 `kernel8.img` 加载到内存地址 `0x80000` 处，并跳转过去执行你的裸机程序。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`树莓派4 64位 裸机 链接脚本 示例`；`Raspberry Pi 4 bare metal linker script`；`aarch64-elf-ld 链接脚本 树莓派`

主要来源：

- [树莓派4裸机基础教程:从hello world开始](https://cloud.tencent.com.cn/developer/article/1700390?from=15425&policyId=undefined&traceId=&frompage=seopage)
- [Raspberry Pi 4B bare metal](https://forums.raspberrypi.com/viewtopic.php?p=1679086&sid=c47d15710d71e0e2b458831868076d1c#p1679086)
- [ruspiro-boot 0.3.2](https://docs.rs/crate/ruspiro-boot/0.3.2/source/src/lib.rs)
- [4 anbox 树莓派_树莓派4裸机基础教程:从hello world开始](https://blog.csdn.net/weixin_36372094/article/details/112191177)
- [RPi4 Bare Metal Stack setting](https://forums.raspberrypi.com/viewtopic.php?p=1977098&sid=db4c9bd1b269d4066a96646994fe7433#p1977098)
- [ruspiro_boot](https://docs.rs/ruspiro-boot/0.3.0/i686-unknown-linux-gnu/ruspiro_boot/#features)
- [树莓派4裸机基础教程：环境搭建](https://cloud.tencent.cn/developer/article/1700338?from=15425&frompage=seopage)
- [(1)制作树莓派裸机操作系统](https://blog.csdn.net/qq_37459242/article/details/119222659)
- [树莓派裸机开发步骤](https://m.elecfans.com/article/1308238.html)
- [Bare metal asm blink led](https://forums.raspberrypi.com/viewtopic.php?p=2040436&sid=4095ddccae43902e9d09e437aa4dfa58#p2040436)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q060"></a>

## #060 · 这个linker.ld文件是手敲的吗？能否通过工具生成？

> 🕒 2026-07-03 21:27 ｜ DeepSeek 消息 ID 119–120 ｜ 主题：链接脚本 linker.ld 详解

**提问**

这个linker.ld文件是手敲的吗？能否通过工具生成？

**回答**

你这个问题问得很关键。直接回答你：**`linker.ld` 文件主要是手写的，但可以通过一些工具和框架来辅助生成，只不过不能完全自动生成。**

在裸机开发中，`linker.ld` 文件是一个**高度定制化**的配置文件，它的核心是告诉链接器如何将代码和数据精确地放置到芯片的内存中。因为不同芯片的内存布局差异很大，所以没有一个通用的自动化工具能自动生成完全符合你要求的脚本[citation:9]。

##### ✍️ 日常开发：手写与模板复用

在真实的开发流程中，这个文件最常见的来源有两个：

*   **参考并修改**：在项目初期，你通常会从一篇**教程、一个开源项目或芯片厂商的示例**中，找到最接近的 `linker.ld` 作为起点[citation:1][citation:8][citation:4]。然后在它的基础上，根据自己芯片的 `datasheet` 修改内存起始地址（`ORIGIN`）和大小（`LENGTH`）[citation:9]。
*   **完全手写**：当需求比较特殊时，就需要根据链接脚本的语法规则从零开始编写。

##### 🛠️ 辅助工具：让手写更高效

虽然没有“一键生成”的工具，但有一些方法可以减轻手写负担：

*   **使用C预处理器（CPP）**：这是将 `linker.ld` 模板化的好方法。你可以像写C代码一样在脚本里定义宏和包含头文件，然后通过 `gcc -E` 这样的命令预处理，生成最终的 `.ld` 文件。这样，一些通用的配置就能复用了。
*   **利用编程语言框架**：在Rust的嵌入式生态中，很多项目通过 `cargo-generate` 这样的工具，从**模板（template）** 生成一个包含基础 `linker.ld` 或 `memory.x` 文件的项目骨架，省去了从零开始的工作[citation:6][citation:9]。

##### 💎 总结

所以，`linker.ld` 本质上是一份需要开发者精心编写的“内存地图”。虽然可以通过工具和框架来辅助生成或复用模板，但它始终需要开发者根据**具体的芯片和项目需求**来定制，无法完全自动生成。这也就意味着，手写或修改它是进入裸机开发的必经之路。

<details>
<summary>🔍 联网检索（4 条查询词 / 8 个来源）</summary>

查询词：`Raspberry Pi 4 "linker.ld" bare metal`；`Rust embedded "linker.ld" generation`；`"cargo-generate" Raspberry Pi linker script`；`GCC linker script generator tool`

主要来源：

- [Raspberry Pi 4B bare metal](https://forums.raspberrypi.com/viewtopic.php?p=1679086&sid=c47d15710d71e0e2b458831868076d1c#p1679086)
- [gcc.gnu.org Git - gcc.git/blob](https://gcc.gnu.org/git/?p=gcc.git;a=blob;f=libgcc/config/t-slibgcc-libgcc;h=4cd3fffcd62b6d2d15a837b410090322ba835cd4;hb=3dba5b2cf2d624921d976670b927bf5330da075c)
- [Bare metal asm blink led](https://forums.raspberrypi.com/viewtopic.php?p=2040436&sid=4095ddccae43902e9d09e437aa4dfa58#p2040436)
- [Tooling - The Embedded Rust Book](https://dev-doc.rust-lang.org/beta/embedded-book/intro/tooling.html)
- [Rust-based Raspberry Piva Raspberry 4 Bare Metal Experience](https://www.programmersought.com/article/957810522402/)
- [Embedded Rust: Adding support for your target microcontroller - Embedded](https://www.embedded.com/embedded-rust-adding-support-for-your-target-microcontroller/?utm_source=eetimes&utm_medium=latestnews&_ga=2.123933066.1671528438.1644750094-1204887681.1597044287)
- [ruspiro-console 0.3.2](https://docs.rs/crate/ruspiro-console/0.3.2/source/Makefile.toml)
- [Raspberry Pi Forums](https://forums.raspberrypi.com/viewtopic.php?t=336898&sid=d32431ee1fad17b82448e29d21cb1f38&view=print)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q061"></a>

## #061 · 有没有教程讲linker.ld中的sections

> 🕒 2026-07-03 21:31 ｜ DeepSeek 消息 ID 121–122 ｜ 主题：链接脚本 linker.ld 详解

**提问**

有没有教程讲linker.ld中的sections

**回答**

`linker.ld` 文件中的 `SECTIONS` 命令是整个链接脚本的核心。网上确实有很多相关的教程，比如，官方文档就有一个非常清晰的基础示例，能帮你快速理解它的工作原理[citation:2]。

简单来说，`SECTIONS` 命令就像一个施工蓝图，它告诉链接器如何将众多输入文件（`.o` 文件）里的不同“零件”（`section`）组合起来，并最终放置到输出文件（可执行文件或内核镜像）的正确位置上[citation:2][citation:5][citation:7]。

##### 📜 `SECTIONS` 命令的基础结构

一个基础的 `SECTIONS` 命令看起来是这样：

```ld
SECTIONS
{
  . = 0x10000;               /* 1. 设置“当前位置” */

  .text : { *(.text) }      /* 2. 定义并填充 .text 输出段 */

  . = 0x8000000;             /* 3. 改变“当前位置” */

  .data : { *(.data) }      /* 4. 定义并填充 .data 输出段 */

  .bss : { *(.bss) }        /* 5. 定义并填充 .bss 输出段 */
}
```

##### 🔍 逐行拆解这个示例

*   **`. = 0x10000;`**：这是一个**位置计数器赋值**。位置计数器（用特殊符号 `.` 表示）是链接器用来跟踪当前输出地址的“指针”[citation:2][citation:5]。这行代码的含义是：将当前位置设置到 `0x10000`。后续的输出段，如果没有特殊指定，就会从这个地址开始放置[citation:9]。

*   **`.text : { *(.text) }`**：这是一个**输出段定义**。
    *   **`.text`**：定义了输出文件中一个名为 `.text` 的段（通常用于存放代码）。
    *   **`:`**：冒号是固定语法。
    *   **`{ *(.text) }`**：花括号里指定了哪些“零件”要放进这个“盒子”。`*` 是一个通配符，匹配所有输入文件[citation:1][citation:2]。`(.text)` 表示匹配输入文件中的 `.text` 段。所以，这整句的意思就是：**把来自所有输入文件（`.o` 文件）的 `.text` 段，合并后放入输出文件的 `.text` 段，位置从 `0x10000` 开始**。

*   **`. = 0x8000000;`**：再次修改位置计数器，将当前地址设为 `0x8000000`。这样，下面的 `.data` 段就会被放置在这个新地址。

*   **`.data : { *(.data) }`**：与 `.text` 类似，将所有输入文件的 `.data` 段（通常存放已初始化的全局变量）合并放入输出文件的 `.data` 段，位置在 `0x8000000`[citation:2]。

*   **`.bss : { *(.bss) }`**：将所有输入文件的 `.bss` 段（通常存放未初始化的全局变量）合并放入输出文件的 `.bss` 段。它的位置紧跟在 `.data` 段之后，由位置计数器自动计算得出[citation:2]。

##### 💡 实战：为树莓派4链接脚本做铺垫

这个基础示例清晰地展示了 `linker.ld` 的核心工作：**决定代码（`.text`）、数据（`.data`）和未初始化数据（`.bss`）在内存中的最终布局**。这与你在为树莓派4构建裸机系统时，需要精确控制内核镜像在内存 `0x80000` 地址加载的需求是完全一致的。

你可以基于这个最简单的模板，通过添加 `ENTRY` 指定入口函数[citation:4]、使用 `MEMORY` 定义内存区域[citation:9]、以及用 `AT` 指定加载地址与运行地址不同（这在从SD卡启动时很常见）等手段，逐步将它完善成真正可用的脚本。

<details>
<summary>🔍 联网检索（3 条查询词 / 9 个来源）</summary>

查询词：`linker.ld sections 教程`；`linker script 教程`；`GNU ld 链接脚本 详解`

主要来源：

- [Input Section Example (LD)](https://www.sourceware.org/binutils/docs-2.39/ld/Input-Section-Example.html)
- [Using LD, the GNU linker](http://csciun1.mala.bc.ca:8080/~pwalsh/teaching/261/261/hc11/doc/ld_3.html)
- [Using LD, the GNU linker](http://ftp.gnu.org/pub/old-gnu/Manuals/ld-2.9.1/html_chapter/ld%5F2.html)
- [Linker scripts](https://users.informatik.haw-hamburg.de/%7Ekrabat/FH-Labor/gnupro/5_GNUPro_Utilities/c_Using_LD/ldLinker_scripts.html)
- [Linux链接脚本学习--lds](https://developer.aliyun.com/article/10603)
- [Using LD, the GNU linker](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_node/ld%5f20.html)
- [LinkerScript脚本秘籍|语法深度解析与实战](https://www.eet-china.com/mp/a346177.html)
- [详解链接器脚本ld文件](https://www.eet-china.com/mp/a300467.html)
- [https://eng.libretexts.org/@api/deki/pages/19886/pdf/5.4%253A%2bAssemble%252FLink%2bScript…](https://eng.libretexts.org/@api/deki/pages/19886/pdf/5.4%253A%2bAssemble%252FLink%2bScript.pdf#1#1)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q062"></a>

## #062 · linker.ld中这一段怎么理解“.text : { KEEP(\*(.text.boot)) \*(.text .tex…

> 🕒 2026-07-03 21:46 ｜ DeepSeek 消息 ID 123–124 ｜ 主题：链接脚本 linker.ld 详解

**提问**

```text
linker.ld中这一段怎么理解“.text : {
        KEEP(*(.text.boot))
        *(.text .text.* .gnu.linkonce.t*)
    }” 这里的KEEP是什么意思？这一段的作用是吧输入的.o文件中的.text段，.text.*和.gnu.linkonce.t*段合并后放入输出文件的.text段，起始地址从0x8000000开始。
```

**回答**

你理解得非常准确！这段 `.text` 输出段的定义，核心作用确实是：**将从所有输入文件（`.o` 文件）中收集来的 `.text`、`.text.*` 和 `.gnu.linkonce.t*` 这些段，合并后放入输出文件的 `.text` 段**。它的起始地址由链接器当前位置决定，在你提供的树莓派4链接脚本上下文中，地址就是 `0x80000`。

而其中的 **`KEEP`**，是这个脚本里一个至关重要的指令，我来帮你详细拆解它的含义。

##### 🤔 `KEEP` 到底在做什么？

`KEEP(*(.text.boot))` 的核心意思是：**即使这个段在最终的程序中没有被显式地“引用”，也要强制保留它，不能将其优化掉。**

这其实是在告诉链接器，你需要对抗一个默认的优化行为——**垃圾回收（Garbage Collection, GC）**。

##### 🧹 链接器的“垃圾回收”机制

现代链接器（如 `aarch64-elf-ld`）在启用 `--gc-sections` 选项后，会执行一个优化步骤：

1.  **从入口点（Entry Point）开始**：链接器会从你指定的入口函数（如 `_start`）开始，追踪它调用了哪些函数，函数又调用了哪些数据和函数。
2.  **标记“存活”的段**：所有能被追踪到的代码和数据段，都会被标记为“存活”。
3.  **丢弃“死”段**：那些没有被任何“存活”代码调用或引用的段，就会被视为“死代码”或“死数据”，在最终的可执行文件或镜像中被**丢弃**。

##### 🚑 为什么裸机程序需要 `KEEP`？

在裸机开发中，`KEEP` 指令通常用于保护一些“入口点”或“特殊用途”的代码，这些代码不会被“常规”代码调用，因此很容易被链接器误判为“死代码”并丢弃。

*   **你的例子：`.text.boot` 段**
    对于树莓派4的裸机程序，`.text.boot` 段通常包含用汇编语言编写的启动代码。这个代码是系统上电后，CPU跳转执行的**第一个入口点**。它不会像常规函数那样被 `main` 或其他函数调用，而是由硬件直接跳转。

*   **如果不用 `KEEP` 会怎样？**
    如果链接器找不到任何代码显式地“调用” `.text.boot` 段中的 `_start` 函数，它就会认为这个段是“死”的，并在优化过程中将其丢弃。结果就是，你的 `kernel8.img` 里将没有任何启动代码，镜像就无法工作了。

**`KEEP` 的作用就是强制将这部分关键的启动代码保留在最终的镜像中，确保内核能够被正常引导。**

##### 💡 总结

简单来说，这个 `.text` 段的定义做了三件事：

1.  **组合**：将所有代码段合并成一个大的代码区。
2.  **布局**：确保在最终生成的 `kernel8.img` 的开头（因为 `. = 0x80000`），并且位于 `.text` 段的起始位置。
3.  **保护**：使用 `KEEP` 确保启动代码（`.text.boot`）这个“孤岛”不会被链接器的优化功能（`--gc-sections`）错误地删除，保证树莓派能顺利启动。

现在你应该理解了 `KEEP` 的必要性：在没有 `KEEP` 的情况下，即使编译链接看似成功，生成的镜像也可能是空的或缺失启动代码的，从而无法正常启动。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q063"></a>

## #063 · 准确的说，是不是将从所有输入文件（.o 文件）中收集来的 .text、.text.\* 和 .gnu.linkonce.t…

> 🕒 2026-07-03 22:00 ｜ DeepSeek 消息 ID 125–126 ｜ 主题：链接脚本 linker.ld 详解

**提问**

```text
准确的说，是不是将从所有输入文件（.o 文件）中收集来的 .text、.text.* 和 .gnu.linkonce.t* 这些段，合并后放入输出文件的 .text.boot 段?
```

**回答**

**不是的**，这里有一个非常关键的概念需要澄清。

**准确的说法是**：将所有输入文件（`.o` 文件）中的 `.text`、`.text.*` 和 `.gnu.linkonce.t*` 段，**合并后放入输出文件的 `.text` 段**（而不是 `.text.boot` 段）。

##### 🔍 逐层拆解这个语法

我们先来看这行代码的结构：
```ld
.text : {
    KEEP(*(.text.boot))
    *(.text .text.* .gnu.linkonce.t*)
}
```

它的结构是： **`输出段名 : { 输入段规则 }`**

*   **`输出段名`**：冒号前面的 **`.text`** 就是**输出文件（最终生成的 `kernel8.elf`）里的段名**。所有花括号里的东西，都会被合并进这个名为 `.text` 的段里。
*   **`输入段规则`**：花括号里的是匹配输入文件（`.o` 文件）的规则，它告诉链接器“要拿哪些东西进来”。

这个花括号里的规则分为两部分：

1.  **`KEEP(*(.text.boot))`**
    *   **作用**：从**所有**输入文件（`*`）中，找到它们各自的 **`.text.boot`** 段（这是输入段），**强制保留**它们，并**放在输出 `.text` 段的最前面**。
    *   **结果**：输入文件里的 `.text.boot`，变成了**输出文件 `.text` 段的一部分**，不再是独立的段。

2.  **`*(.text .text.* .gnu.linkonce.t*)`**
    *   **作用**：从**所有**输入文件（`*`）中，找到它们各自的 **`.text`**、**`.text.*`** 和 **`.gnu.linkonce.t*`** 段，按顺序放入输出文件的 `.text` 段，紧跟在 `KEEP` 那部分后面。

##### 💡 一个形象的类比

你可以把链接器想象成一个“行李打包员”：

*   **输出段名 `.text`**：一个**大行李箱**。
*   **输入文件 `.text.boot`**：一个**小背包**（里面装的是启动代码）。
*   **输入文件 `.text`**、**`.text.*`**：**其他小包**（里面装的是普通代码）。

`KEEP` 指令相当于行李员在打包时，会在这个小背包上贴一个“**贵重物品，严禁丢弃**”的标签（防止丢失）。然后，他会把这个小背包和所有其他小包，一起**放进行李箱 `（.text）` 的最底层（最前面）**。最后，这个大行李箱就叫 `.text`。

##### 🎯 为什么 `.text.boot` 能放在 `.text` 最前面？

这行 `.text : { ... }` 的定义顺序决定了这一点。在链接脚本中，输出段内部各个输入规则的**顺序，决定了它们在内存中的排列顺序**。因此，`KEEP(*(.text.boot))` 被写在最前面，所以它在输出文件的 `.text` 段中就是首部位置，确保启动代码位于镜像的最开头。

你在之前的验证中看到 `.text.boot` 部分，指的就是这个被放置在 `.text` 段最前端的、被保护起来的启动代码。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q064"></a>

## #064 · 这一段：“ .rodata : { \*(.rodata .rodata.\* .gnu.linkonce.r\*) }”意思…

> 🕒 2026-07-03 22:11 ｜ DeepSeek 消息 ID 127–128 ｜ 主题：链接脚本 linker.ld 详解

**提问**

```text
这一段：“
    .rodata : {
        *(.rodata .rodata.* .gnu.linkonce.r*)
    }”意思是讲输入文件（.o)中的.rodata.rodata.*段和.gnu.linkonce.r*段放入输出文件的.rodata段，.rodata段的开始是紧挨着.text段的结束。
```

**回答**

**你的理解完全正确！** 你不仅准确掌握了 `.rodata` 段的语法，还对链接脚本的**核心执行逻辑**——**顺序决定地址**——有了很清晰的认识。

##### 1. 语法解析：没错，这是合并操作
你的理解非常精准：

-   **输出段**：这行代码定义了一个名为 `.rodata` 的**输出段**。
-   **输入段**：它将来自所有输入文件（`.o` 文件）的 `.rodata`、`.rodata.*` 和 `.gnu.linkonce.r*` 这些**输入段**，全部收集并合并到这个输出的 `.rodata` 段中。

##### 2. 内存布局：完全正确，是紧挨着的
你提到“`.rodata` 段的开始是紧挨着 `.text` 段的结束”，这个结论**完全正确**，这正是链接脚本 **“顺序即地址”** 这一核心原则的体现。

在你的 `linker.ld` 脚本中，如果你是这样编写的：

```ld
SECTIONS {
    . = 0x80000;

    .text : {
        KEEP(*(.text.boot))
        *(.text .text.* .gnu.linkonce.t*)
    }

    .rodata : {
        *(.rodata .rodata.* .gnu.linkonce.r*)
    }

    /* ... 后续段 ... */
}
```

链接器在执行时，会像流水线一样逐行处理：

1.  首先，将 `.text` 段的**所有内容**，从 `0x80000` 这个地址开始，连续地摆放下去。
2.  当 `.text` 段的内容全部摆放完毕后，**当前位置（`.`）** 就停留在了 `.text` 段结束的位置。
3.  紧接着，开始处理 `.rodata` 段。因为中间没有插入任何 `. = 某地址` 的跳转指令，`.rodata` 段会**自然而然地、无缝地**从当前位置开始摆放。

所以，最终生成的内核镜像中，内存布局就是一个连续的整体：

```text
地址 0x80000:
    ┌─────────────────┐  ← .text段起始
    │   .text.boot    │
    ├─────────────────┤
    │   .text         │
    │   .text.*       │
    └─────────────────┘  ← .text段结束，.rodata段起始
    ┌─────────────────┐
    │   .rodata       │
    │   .rodata.*     │
    └─────────────────┘  ← .rodata段结束
    ┌─────────────────┐
    │   .data         │  ← 后续段继续...
    └─────────────────┘
```

##### 3. 为什么 `.rodata` 要紧挨着 `.text`？
这种“紧挨着”的布局，在裸机程序中能最大限度地**节约内存**。所有的代码和只读数据都被高效地、连续地打包在一起，没有任何内存空隙，这对于资源受限的嵌入式系统来说至关重要。

恭喜你，你对链接脚本的理解又深入了一层。按这个思路往下看，`.data` 和 `.bss` 段的布局逻辑也就触类旁通了。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q065"></a>

## #065 · 这一段“.bss (NOLOAD) : { . = ALIGN(16); __bss_start = .; \*(.bss…

> 🕒 2026-07-03 22:16 ｜ DeepSeek 消息 ID 129–130 ｜ 主题：链接脚本 linker.ld 详解

**提问**

```text
这一段“.bss (NOLOAD) : {
        . = ALIGN(16);
        __bss_start = .;
        *(.bss .bss.*)
        *(COMMON)
        __bss_end = .;
    }”怎么理解？
```

**回答**

这段 `.bss` 段的定义是链接脚本里一个非常经典的写法。你之前对 `.rodata` 的理解很到位，现在可以把这个理解也应用到 `.bss` 段上，只需要再掌握它的三个特殊之处。

##### 🎯 核心功能：定义 BSS 段并记录地址

`.bss` 段用于存放未初始化的全局变量和静态变量。这段脚本的核心作用是：**收集所有未初始化的数据，将它们规划到内存的一个连续区域，并精确记录下这片区域的起始和结束地址**，以供启动代码在运行 `main` 函数前将其清零。

##### 🔍 逐行拆解特殊语法

我们来详细看看脚本中的每一个关键点：

1.  **`(NOLOAD)`**：
    *   这是一个**段属性标记**。它的作用是告诉链接器：“这个段的内容不需要在最终的镜像文件中占用实际空间，它的数据在程序加载或运行时才在内存中生成，且初值应为零。”
    *   你之前看到的 `.text` 和 `.rodata` 段，它们的内容必须被烧录进镜像文件。而 `.bss` 段只记录需要预留的**内存区域**，这是一个重要的区别。

2.  **`. = ALIGN(16);`**：
    *   这是一个**对齐指令**。它的意思是：将当前位置（`.`）向上对齐到下一个 **16 字节**的整数倍地址。
    *   这确保了 BSS 段在内存中是从一个合适的对齐地址开始的，对后续的数据访问性能至关重要。

3.  **`__bss_start = .;` 和 `__bss_end = .;`**：
    *   这是**符号赋值语句**。它创建了两个在汇编和C语言中都能访问的全局符号（类似于变量）。
    *   `__bss_start` 被设置为当前地址（`.bss` 段的起始地址），`__bss_end` 被设置为整个 BSS 段结束时的地址。
    *   **启动代码的作用**：C 运行时通常要求未初始化的全局变量在 `main` 函数运行前其值为零。你的汇编启动代码（在 `_start` 中）会读取这两个符号，然后使用它们作为地址范围，用一个循环将这片内存全部写入 0。这正是 `__bss_start` 和 `__bss_end` 被导出的意义。

##### 💡 与 `.data` 段的协作分工

为了帮助你更清晰地理解，需要补充说明一下 `.bss` 段和 `.data` 段的分工：

*   **`.data` 段**：存放**已初始化**的全局变量。这些变量有预设的初值，需要占用镜像文件空间来保存这些初始数据。
*   **`.bss` 段**：存放**未初始化**的全局变量。它们不需要在镜像文件中占用空间，只需在内存中预留出一块区域，并在启动时统一清零。

##### 💎 总结

简单来说，这段 `.bss` 段脚本做的主要是两件事：

1.  **内存布局规划**：它在内存中规划出一片区域，用来存放所有未初始化的全局变量。
2.  **提供清零依据**：它通过 `__bss_start` 和 `__bss_end` 这两个符号，为启动代码提供这片区域的精确范围，以便系统启动时能将这片内存全部清零。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q066"></a>

## #066 · \*(COMMON)是什么意思？

> 🕒 2026-07-03 22:24 ｜ DeepSeek 消息 ID 131–132 ｜ 主题：链接脚本 linker.ld 详解

**提问**

```text
*(COMMON)是什么意思？
```

**回答**

`*(COMMON)` 是链接脚本中一个非常独特且重要的指令，它专门用于处理**未初始化的全局变量**在链接过程中的特殊安排。

##### 📦 `COMMON` 段是什么？

`COMMON` 是一个**伪段**，它并不像 `.text` 或 `.data` 那样真实存在于目标文件（`.o`）中，而是编译器（GCC）对**未初始化全局变量**的一种**临时存放方式**。

在 C 语言中，当你在文件作用域写下：
```c
int global_var;  // 未初始化的全局变量
```
编译器不会立刻决定把它放在 `.bss` 段，而是先将其标记为 `COMMON` 类型，放在一个临时区域。这个决定要等到**链接**阶段才会最终做出。

##### 🧩 `*(COMMON)` 的作用

`*(COMMON)` 指令的核心作用是：**将所有输入文件中所有标记为 `COMMON` 的未初始化全局变量，都收集并放入当前输出文件的 `.bss` 段中**。

具体来看：
- `*`：通配符，匹配所有输入文件。
- `(COMMON)`：选择输入文件中所有标记为 `COMMON` 的符号。

这行指令确保了所有分散在不同 `.o` 文件中的未初始化全局变量，最终都能被合并到输出文件的 `.bss` 段中，形成一个连续的内存区域。

##### 🤔 为什么需要单独处理 `COMMON`？

GCC 把未初始化的全局变量放在 `COMMON` 而不是直接放入 `.bss`，主要是为了支持一种历史悠久的 C 语言特性——**“暂定定义”（Tentative Definition）**。

简单来说，它允许你在多个文件中声明同一个全局变量，而不会立即报错。链接器在最终链接时，会将所有同名的 `COMMON` 符号合并，确保只分配一份内存。这正是 `*(COMMON)` 指令存在的意义：在链接的最后阶段，将所有 `COMMON` 符号收集起来，合并分配到一个最终的 `.bss` 区域。

##### 💎 总结

- **`*(.bss)`**：负责将那些已经被明确标记为 `.bss` 的未初始化数据放入 BSS 段。
- **`*(COMMON)`**：负责将 GCC 暂时放在 `COMMON` 区域的未初始化全局变量，也最终放入 BSS 段。

两者结合，才能确保所有类型的未初始化全局变量都被正确收集，并统一在 `.bss` 段中完成内存分配。所以，在链接脚本的 `.bss` 段定义中，同时包含 `*(.bss)` 和 `*(COMMON)` 是一种非常标准的写法，旨在确保所有未初始化的全局数据都能被正确处理。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---
