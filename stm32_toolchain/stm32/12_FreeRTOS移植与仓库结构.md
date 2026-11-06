<a id="top"></a>

# FreeRTOS 移植与仓库结构

> 归档自 DeepSeek 分享对话：<https://chat.deepseek.com/share/ytrbqq4h7dvcvhajt9>  
> 本主题共 **5 轮**问答（原始全局编号 #091–#104），时间 2026-07-31 ~ 2026-08-18。  
> 所属分组：**STM32 开发**  
> 提问与回答均为原始内容；回答内小标题层级整体下沉 2 级，`[reference:N]` 占位符已尽量还原为来源链接。  
> [« 返回总索引](../README.md)

## 本主题目录

- [#091 FreeRTOS支持STM32F407ZGT6开发板吗？](#q091)
- [#093 现在Github上的freertos项目的目录结构和以前的不一样了，原来freertos中的demo目录现在到哪里去了？](#q093)
- [#094 是git clone https://github.com/FreeRTOS/FreeRTOS 克隆主仓库吗？](#q094)
- [#095 子模块的形式来引用是什么意思？](#q095)
- [#104 现在上电成功后，下一步是否将RTOS移植到开发板上让它跑起来？](#q104)

---

<a id="q091"></a>

## #091 · FreeRTOS支持STM32F407ZGT6开发板吗？

> 🕒 2026-07-31 15:10 ｜ DeepSeek 消息 ID 181–182 ｜ 主题：FreeRTOS 移植与仓库结构

**提问**

FreeRTOS支持STM32F407ZGT6开发板吗？

**回答**

**支持的，而且 FreeRTOS 在 STM32F407ZGT6 上的应用非常成熟和广泛。**

这可以说是目前学习嵌入式实时操作系统最经典、资料最丰富的组合之一。官方和社区都提供了完善的支持，你可以通过几种不同的方式快速上手。

##### 🧩 成熟的生态与资料

*   **广泛的生态支持**：STM32F407ZGT6 是许多开发板（如正点原子探索者）的核心，围绕它的学习资料和实践项目非常多。FreeRTOS 作为一个流行的 RTOS，在 STM32F407 上的移植和使用已经非常标准化[citation:3][citation:12]。
*   **丰富的移植教程**：你可以轻松找到大量详细的移植教程，手把手教你如何将 FreeRTOS 源码添加到 STM32F407 的基础工程中，包括配置内核文件、处理中断服务函数等步骤[citation:2][citation:7][citation:9]。
*   **实际项目的验证**：许多实际项目都基于这个组合。例如，有项目使用 **STM32F407ZGT6 + FreeRTOS** 实现多传感器数据采集同步系统，也有项目将其与 **LwIP** 协议栈结合，实现网络通信功能[citation:6]。这证明了其组合的稳定性和实用性。

##### 🛠️ 两种主流开发方式

你可以根据自己的偏好，选择以下任何一种方式来开始：

1.  **图形化配置：STM32CubeMX + Keil/STM32CubeIDE（推荐新手）**
    *   **原理**：使用 ST 官方的图形化配置工具 STM32CubeMX，你只需要在图形界面中勾选启用 FreeRTOS，它就会自动为你生成一个包含 FreeRTOS 内核的初始化工程。
    *   **优点**：省去了手动复制源码和修改配置文件的繁琐步骤，能让你更聚焦于应用逻辑的开发[citation:3][citation:12]。许多教材也推荐这种高效的开发方式[citation:1]。

2.  **手动移植：FreeRTOS源码 + 基础工程（学习内核）**
    *   **原理**：从 FreeRTOS 官网下载源码包，手动将核心文件（`tasks.c`, `queue.c`, `list.c` 等）和与硬件相关的可移植文件（`port.c`, `heap_4.c` 等）复制到你的 Keil 或 IAR 基础工程中，并解决配置冲突[citation:2][citation:7][citation:10]。
    *   **优点**：能让你更深入地理解 FreeRTOS 的源码结构、内存管理方式（`heap_x.c`）以及如何与 Cortex-M4 内核交互。

##### 🔧 性能说明
STM32F407ZGT6 搭载的 Cortex-M4 内核，其处理能力和资源（1MB Flash, 192KB RAM）对于运行 FreeRTOS 以及创建多个任务、使用信号量、消息队列等机制来说是绰绰有余的[citation:3][citation:5]。FreeRTOS 本身非常轻量，对资源的消耗很小。

你完全可以放心在你的那块 STM32F407ZGT6 开发板上，使用 FreeRTOS 开发项目。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`"FreeRTOS" "STM32F407ZGT6" 支持`；`FreeRTOS STM32F407 移植`；`STM32F407ZGT6 FreeRTOS 例程`

主要来源：

- [清华大学出版社--图书详情](http://tup.com.cn/booksCenter/book_10812301.html)
- [FreeRTOS | STM32F407 FreeRTOS移植(第十四天)](https://cloud.tencent.com.cn/developer/article/2460148?policyId=1003)
- [STM32Cubemx实战：从零到一构建基于STM32F407ZGT6的FreeRTOS多任务应用框架](https://blog.csdn.net/weixin_29193259/article/details/158864285)
- [FreeRTOS实时操作系统(架构移植与开发)/计算机科学与技术丛书](https://www.yuntaigo.com/book.action?recordid=bmJsbnpoaGM5Nzg3MzAyNjg3MTQ2)
- [GitHub - soda-code/STM32F407_Creat · GitHub](https://github.com/soda-code/STM32F407_Creat)
- [【STM32F4】STM32F407_FreeRTOS_LwIP调试详解一：Cube_MX生成工程](https://club.gizwits.com/thread-8912-1-4.html)
- [FreeRTOS移植到stm32f407的详细步骤与错误解决-开发者社区-阿里云](https://developer.aliyun.com/article/1009448)
- [STM32F407ZGT6移植freeRTOS](https://blog.csdn.net/m0_58341340/article/details/145887974)
- [如何利用官方移植例子，创建一个自己demo工程](http://news.eeworld.com.cn/mcu/ic661586.html)
- [具有可调高精度的多模式经颅电刺激设备设计](https://d.wanfangdata.com.cn/periodical/CiBQZXJpb2RpY2FsQ0hJU29scjkyMDI2MDYwMjE2MjUxORISemd5eHdseHp6MjAyNTA5MDEzGghlejVlZjZlbQ%3D%3D)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](../README.md)

---

<a id="q093"></a>

## #093 · 现在Github上的freertos项目的目录结构和以前的不一样了，原来freertos中的demo目录现在到哪里去了？

> 🕒 2026-07-31 15:27 ｜ DeepSeek 消息 ID 185–186 ｜ 主题：FreeRTOS 移植与仓库结构

**提问**

现在Github上的freertos项目的目录结构和以前的不一样了，原来freertos中的demo目录现在到哪里去了？

**回答**

是的，这个变化确实让许多开发者感到困惑。`demo` 目录并没有消失，而是随着 FreeRTOS 在 GitHub 上的大型重构，**被移动到了主仓库的顶层目录下，并被拆分成多个独立的库仓库**，采用子模块（submodule）的形式来引用。

##### 📂 新旧结构对比

简单来说，旧结构是把所有东西都放在一个大仓库里，而新结构则是“组件化”和“模块化”的。

| 旧结构 (单体仓库) | 新结构 (模块化仓库) |
| :--- | :--- |
| `FreeRTOS/Demo` 目录包含所有示例 | `FreeRTOS/FreeRTOS` 主仓库使用子模块 |
| Demo 与内核源码在同一个仓库中 | 内核源码在 `FreeRTOS/FreeRTOS-Kernel` 独立仓库[citation:8] |
| 通过目录名区分不同平台示例 | Demo 项目在主仓库中，但**内核源码通过子模块引用**[citation:1][citation:8] |

##### 🔍 如何找到 `demo` 内容？

现在的思路是：你不再需要一次性下载所有平台的演示，而是可以根据自己的目标平台，在官方文档指引下找到对应的演示项目。

1.  **查找主仓库的顶层目录**：如果你克隆了 `FreeRTOS/FreeRTOS` 这个主仓库，会发现项目根目录下确实有 `FreeRTOS/Demo` 和 `FreeRTOS/Source` 这样的路径[citation:10][citation:4]。这个 `Demo` 目录就是目前存放预配置项目的位置，其**子目录名称会标识目标设备和编译器**[citation:2][citation:10]。例如，与 STM32F407 相关的演示可能会在 `FreeRTOS/Demo/CORTEX_M4F_STM32F407ZG-SK` 这样的目录下。

2.  **注意“内核”与“演示”的分离**：随着版本演进，内核源码（`tasks.c`, `queue.c` 等）被独立到了 `FreeRTOS/FreeRTOS-Kernel` 仓库[citation:8]。主仓库通过 **Git 子模块（submodule）** 来引用它[citation:1]。这意味着，如果你直接下载内核仓库（`FreeRTOS-Kernel`），里面是没有 `Demo` 目录的。

3.  **优先查阅官方文档**：由于结构和版本变化较快，最可靠的方式是查阅你下载的 FreeRTOS 版本中的 `README.md` 文件，或者访问 FreeRTOS 官网的 **“Source Code Organization”** 页面。这些文档会提供最准确的目录结构说明和查找示例的方法[citation:4][citation:10]。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`FreeRTOS GitHub repository directory structure changes`；`FreeRTOS demos directory location`；`FreeRTOS GitHub repository 2026 directory layout`

主要来源：

- [FreeRTOS FAQ - GitHub Repository Structure &amp; Versioning - FreeRTOS™](https://key.freertos.org/Why-FreeRTOS/FAQs/Github-repository-structure-and-versioning##how-are-freertos-libraries-versioned)
- [FreeRTOS 演示应用程序 - FreeRTOS™](https://key.freertos.org/zh-cn-cmn-s/Documentation/02-Kernel/03-Supported-devices/04-Demos/01-Demo-overview)
- [FreeRTOS 常见问题 - GitHub 存储库结构 &amp; 版本控制 - FreeRTOS™](https://wina.freertos.org/zh-cn-cmn-s/Why-FreeRTOS/FAQs/Github-repository-structure-and-versioning#freertos-git-%e5%ad%98%e5%82%a8%e5%ba%93%e6%98%af%e5%a6%82%e4%bd%95%e6%9e%84%e5%bb%ba%e7%9a%84)
- [FreeRTOS/FreeRTOS/README.md at 91659446648a00f24b0b8af6327230658cc65c07 · FreeRTOS/FreeRTO…](https://github.com/FreeRTOS/FreeRTOS/blob/91659446/FreeRTOS/README.md?plain=1#L11-L23)
- [Source Code Organization - FreeRTOS™](https://2fwww.freertos.org/Documentation/03-Libraries/05-FreeRTOS-labs/04-FreeRTOS-plus-FAT/02-Source-code-organization)
- [https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/41fe3f0da43a329e8c942318322453…](https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/41fe3f0da43a329e8c9423183224536ab8805538/doc_source/portingguide/porting-migration.md)
- [https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/8a4dffcce43f5669039ce502ede063…](https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/8a4dffcce43f5669039ce502ede06364b8bbc696/doc_source/userguide/freertos-next-steps.md)
- [c6636f465f2bc70a31963356e549ebf514f892c3 - third_party/github/FreeRTOS/FreeRTOS-Kernel](https://pigweed.googlesource.com/third_party/github/FreeRTOS/FreeRTOS-Kernel/+/c6636f465f2bc70a31963356e549ebf514f892c3)
- [FreeRTOS - A description of the FreeRTOS demo tasks that are common to all demos, and how…](http://www.openrtos.net/a00102.html)
- [源组织 - FreeRTOS™](https://shop.nxp.freertos.org/zh-cn-cmn-s/Documentation/02-Kernel/06-Coding-guidelines/01-Source-code-organization#%e5%9f%ba%e6%9c%ac%e7%9b%ae%e5%bd%95%e7%bb%93%e6%9e%84)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](../README.md)

---

<a id="q094"></a>

## #094 · 是git clone https://github.com/FreeRTOS/FreeRTOS 克隆主仓库吗？

> 🕒 2026-07-31 15:36 ｜ DeepSeek 消息 ID 187–188 ｜ 主题：FreeRTOS 移植与仓库结构

**提问**

是git clone https://github.com/FreeRTOS/FreeRTOS 克隆主仓库吗？

**回答**

是的，克隆主仓库 `git clone https://github.com/FreeRTOS/FreeRTOS.git` 是获取 FreeRTOS 源码与 Demo 项目的正确入口。

##### 🔍 仓库结构与克隆方式

这个主仓库是一个“包仓库”，它通过 Git 子模块（submodule）引用了各个独立的组件库（包括内核源码 `FreeRTOS-Kernel`）[citation:5][citation:6][citation:12]。因此，为了完整获取所有依赖，克隆时需要带上 `--recurse-submodules` 参数：

```bash
git clone https://github.com/FreeRTOS/FreeRTOS.git --recurse-submodules
```

如果已经克隆但忘记带上该参数，可以在仓库目录内执行以下命令补全[citation:5][citation:12]：
```bash
git submodule update --init --recursive
```

##### 📂 主仓库的目录结构

克隆完成后，你之前关心的 `Demo` 目录就在主仓库的以下路径中[citation:5][citation:10][citation:12]：

*   **内核 Demo 项目**：`FreeRTOS/Demo` 目录下，包含了针对不同硬件平台和编译器的预配置示例项目。这就是你之前问的 demo 目录现在的所在位置[citation:5][citation:12]。
*   **内核源码**：`FreeRTOS/Source` 目录通过子模块链接了内核源码仓库 (`FreeRTOS-Kernel`) [citation:5][citation:12]。
*   **附加组件**：`FreeRTOS-Plus` 目录包含了其他 FreeRTOS 组件库（如网络协议栈）及其对应的示例项目[citation:5][citation:10][citation:12]。

简而言之，直接克隆主仓库并按需使用 `--recurse-submodules` 选项，就能得到一个包含所有必要源码和丰富示例的完整工程。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`"FreeRTOS" "FreeRTOS-Kernel" "Demo" 目录 结构`；`FreeRTOS GitHub 仓库 克隆 主仓库`；`FreeRTOS 目录结构 变化 说明`

主要来源：

- [FreeRTOS/Demo/RX100-RSK_Renesas_e2studio/CreateProjectDirectoryStructure.bat - third_party…](https://pigweed.googlesource.com/third_party/github/FreeRTOS/FreeRTOS-Kernel/+/99d4f2c454de45b6b937bd7d934d55dc6a3d52b2/FreeRTOS/Demo/RX100-RSK_Renesas_e2studio/CreateProjectDirectoryStructure.bat)
- [GitHub - sophgo/freertos: 'Classic' FreeRTOS distribution. Started as Git clone of FreeRTO…](https://github.com/sophgo/freertos)
- [https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/41fe3f0da43a329e8c942318322453…](https://raw.githubusercontent.com/awsdocs/aws-freertos-docs/41fe3f0da43a329e8c9423183224536ab8805538/doc_source/portingguide/porting-migration.md)
- [FreeRTOS/Demo/CORTEX_R4_RM48_CCS5/CreateProjectDirectoryStructure.bat - third_party/github…](https://pigweed.googlesource.com/third_party/github/FreeRTOS/FreeRTOS-Kernel/+/dbe0ae2bffc79c8f0545abecbae8a16476585d52/FreeRTOS/Demo/CORTEX_R4_RM48_CCS5/CreateProjectDirectoryStructure.bat)
- [GitHub - felipebetancur/FreeRTOS-4: 'Classic' FreeRTOS distribution. Started as Git clone…](https://github.com/felipebetancur/FreeRTOS-4)
- [FreeRTOS FAQ - GitHub Repository Structure &amp; Versioning - FreeRTOS™](https://shop.nxp.freertos.org/Why-FreeRTOS/FAQs/Github-repository-structure-and-versioning#how-are-freertos-git-repositories-structured)
- [FreeRTOS的移植（STM32F103）](https://cloud.tencent.cn/developer/article/2531336?policyId=1004)
- [https://egit.irs.uni-stuttgart.de/eive/fsfw/commit/10e81fdfb88e116538f8c7c70a19e6bbe3a54a7…](https://egit.irs.uni-stuttgart.de/eive/fsfw/commit/10e81fdfb88e116538f8c7c70a19e6bbe3a54a70.diff)
- [FreeRTOS-Kernel](https://gitea.com/tomosnelli/FreeRTOS-Kernel/src/tag/V5.0.0)
- [Source Organization [Getting Started]](http://www.openrtos.org/a00017.html#CreateYourOwn)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](../README.md)

---

<a id="q095"></a>

## #095 · 子模块的形式来引用是什么意思？

> 🕒 2026-07-31 15:44 ｜ DeepSeek 消息 ID 189–190 ｜ 主题：FreeRTOS 移植与仓库结构

**提问**

子模块的形式来引用是什么意思？

**回答**

子模块（submodule）的形式来引用，简单来说，就是允许一个 Git 仓库**作为子目录“嵌套”在另一个 Git 仓库中**，同时这两个仓库各自独立维护自己的版本历史。

你可以把它想象成一个“套娃”结构：主仓库（FreeRTOS）是一个盒子，里面放着的内核仓库（FreeRTOS-Kernel）是另一个独立的、完整的小盒子。

##### 🤔 为什么要这样设计？

这主要是为了解决**依赖管理**的问题。以前 FreeRTOS 的代码都在一个大仓库里，所有内容都耦合在一起。当开发者想为大仓库添加新功能或修复 bug 时，任何改动都牵扯到整个项目，提交历史会变得杂乱。而且如果多个不同的项目需要引用同一个组件（比如内核），就必须复制一份代码，难以维护统一的版本。

采用子模块后，**每个模块（如内核、网络协议栈等）都有自己独立的仓库、独立的版本号、独立的维护者**。主仓库只负责把它们“拼”在一起。如果只想使用内核源码，可以直接克隆 `FreeRTOS-Kernel` 仓库，而不需要下载整个 FreeRTOS 项目。

##### 💡 对于普通用户的直接影响

当你克隆 FreeRTOS 主仓库时，**默认情况下，这些子模块的目录是空的**。你必须执行额外的命令来把它们“拉”进来。

一个形象的类比是：
*   主仓库就像一份**购物清单**，上面写了“需要 1 个 A 模块，1 个 B 模块”。
*   子模块命令（`git submodule update --init`）就像**去仓库取出这些模块**，然后把它们放到购物清单指定的位置。

##### 🛠️ 实际操作的关键

为了完整获取 FreeRTOS 的全部源码，推荐使用这条“一揽子”命令：
```bash
git clone --recurse-submodules https://github.com/FreeRTOS/FreeRTOS.git
```
`--recurse-submodules` 参数的作用，就是在克隆主仓库的同时，自动初始化并更新所有子模块。如果你已经克隆了主仓库但没有带上这个参数，可以用 `git submodule update --init --recursive` 来补全。

[↑ 回到顶部](#top) ｜ [« 返回总索引](../README.md)

---

<a id="q104"></a>

## #104 · 现在上电成功后，下一步是否将RTOS移植到开发板上让它跑起来？

> 🕒 2026-08-18 14:28 ｜ DeepSeek 消息 ID 207–208 ｜ 主题：FreeRTOS 移植与仓库结构

**提问**

现在上电成功后，下一步是否将RTOS移植到开发板上让它跑起来？

**回答**

现在开发板已经成功上电，确实可以考虑下一步将 FreeRTOS 移植上去，让系统跑起来了。对 STM32F407 这块板子来说，移植 FreeRTOS 是一个非常成熟、资料丰富的操作[citation:1][citation:4][citation:10]。

根据你的开发环境和偏好，主要有以下三种主流的移植方式：

| 移植方式 | 核心操作 | 优点 | 适合场景 |
| :--- | :--- | :--- | :--- |
| **STM32CubeMX 图形化配置** | 在 CubeMX 的“Middleware”中勾选 FreeRTOS 选项，配置参数后生成代码[citation:2][citation:5][citation:7] | **最简单、最现代**，大部分移植工作由工具自动完成，极大降低入门门槛 | 新手入门、追求开发效率的项目 |
| **手动移植源码** | 从官网下载 FreeRTOS 源码包，手动将内核源码（`Source`文件夹）、移植层代码（`portable`）和配置文件（`FreeRTOSConfig.h`）添加到你的工程中[citation:1][citation:4][citation:10] | **学习最深入**，能帮你理清 FreeRTOS 的架构，理解每一个文件的作用 | 深入学习RTOS原理，或需要对系统进行深度定制的场景 |
| **STM32CubeIDE 直接创建** | 在 STM32CubeIDE 的新建项目向导中，直接选择使用 FreeRTOS 作为中间件 | **集成度最高**，CubeIDE 内部集成了 CubeMX 的配置能力，流程非常顺畅[citation:7] | 使用 STM32CubeIDE 作为主力开发环境的用户 |

##### 🚀 如果是新手，最推荐的方式
从零开始手动移植虽然能学到很多东西，但对新手来说可能会遇到不少“坑”，比如处理重复定义的中断函数（`SysTick_Handler`等）和关闭未实现的钩子函数（`vApplication...Hook`）宏定义[citation:10]。因此，对于第一次尝试，强烈建议从 **STM32CubeMX 图形化配置**开始[citation:2][citation:7]。

这个方法的流程大致是：
1.  在 CubeMX 中为你的 STM32F407 配置好基础的时钟和外设（比如一个用来点灯的 GPIO）。
2.  在 **Middleware** 标签页里，使能 **FREERTOS** 功能。
3.  配置任务（Task）的数量、栈大小和优先级[citation:2]。通常可以创建两个简单的点灯任务来验证。
4.  生成代码。CubeMX 会自动将 FreeRTOS 的内核源码和 HAL 库结合好。
5.  在生成的 `main.c` 和 `freertos.c` 文件中，填入你自己的任务处理函数（比如让 LED 每隔 1 秒翻转一次）[citation:1][citation:2]。

> **一个小提醒**：启用 FreeRTOS 时，CubeMX 通常会提示`SysTick`时钟有冲突，因为 HAL 库的 `HAL_Delay()` 也用了它。这时候按 CubeMX 的建议，为 HAL 分配一个非关键的定时器（比如 TIM6）作为时基就行，它会自动配置好，不用自己操心[citation:7]。

##### 💎 总结
上电成功是很好的开始。下一步从 **STM32CubeMX** 配合 FreeRTOS 的方式入手，会是最流畅、障碍最少的学习路径。等你通过这种“点点鼠标”的方式跑通了一个多任务程序，再回头去看手动移植的源码，理解起来会快很多。如果在配置过程中遇到具体的编译或链接报错，可以带着具体错误信息再来问我。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`STM32F407 移植 FreeRTOS 步骤`；`STM32CubeMX FreeRTOS 生成代码`；`STM32F407 开发板 烧录 程序`

主要来源：

- [FreeRTOS | STM32F407 FreeRTOS移植(第十四天) - Qt历险记](https://cloud.tencent.com.cn/developer/article/2460148?policyId=1003#1)
- [【星允派 NEBULA PI】11:cube创建RTOS项目-单片机/MCU论坛-芯查查](https://www.xcc.com/planet/post/7029)
- [4. 如何用串口下载程序¶ - 4. 如何用串口下载程序¶](https://doc.embedfire.com/mcu/stm32/f407batianhu/std/zh/latest/book/ISP_download.html#1)
- [GitHub - LinmingZhou234/Test-FreeRTOS-V9: 测试FreeRTOS以及移植模板和中文注释 · GitHub - GitHub - Linmin…](https://github.com/LinmingZhou234/Test-FreeRTOS-V9#1)
- [STM32H7S3L8+RTOS [Copy link] - X](https://en.eeworld.com.cn/bbs/forum.php?mod=viewthread&tid=1347004&page=1#1)
- [DAP仿真器与串口下载程序¶ - STM32开发板必读说明](https://doc.embedfire.com/stm32_products/must_read/zh/latest/doc/quickstart/DAP/DAP.html#id11#1)
- [03. FreeRTOS on STM32 with CubeIDE - Skip to content](https://github.com/openhorizonrobotics/E-2-ES/wiki/03.-FreeRTOS-on-STM32-with-CubeIDE#1)
- [GitHub - abdalrahman01/FreeRTOS: FreeRTOS on STM32F4 with STM32CubeMX, Keil uVision 5 and…](https://github.com/abdalrahman01/FreeRTOS#1)
- [GitHub - Arisudan/STM32-Using-Python: Blinking LEDs with MicroPython on STM32! 🚀 This repo…](https://github.com/Arisudan/STM32-Using-Python#1)
- [FreeRTOS移植到stm32f407的详细步骤与错误解决-开发者社区-阿里云 - FreeRTOS-stm32f407移植](https://developer.aliyun.com/article/1009448#1)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](../README.md)

---
