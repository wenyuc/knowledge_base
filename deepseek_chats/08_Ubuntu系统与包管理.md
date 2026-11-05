<a id="top"></a>

# Ubuntu 系统与软件包管理（man / apt / unminimize）

> 归档自 DeepSeek 分享对话：<https://chat.deepseek.com/share/4fgpffcy16vh4nzis8>  
> 本主题共 **9 轮**问答（原始全局编号 #054–#128），时间 2026-09-29 ~ 2026-10-16。  
> 提问与回答均为原始内容；回答内的小标题层级整体下沉 2 级以适配本文件结构，`[reference:N]` 引用占位符已尽量还原为来源链接。  
> [« 返回总索引](./README.md)

## 本主题目录

- [#054 如何查询apt 安装的build-essential的路径](#q054)
- [#062 通过apt install build-essential的时候会不会安装gdb？](#q062)
- [#072 我的Ubuntu中man命令不能用，怎么办？](#q072)
- [#073 $ man --version This system has been minimized by removing p…](#q073)
- [#074 unminimize之后会把我之前安装的覆盖掉吗](#q074)
- [#075 编译时-lssl报错，没有libssl.a的包](#q075)
- [#076 怎样通过apt检查是否安装了libssl-dev](#q076)
- [#077 如何查看安装包的development libraries, header files, and manpages](#q077)
- [#128 在Ubuntu Linux上如何查机器物理内存的大小？](#q128)

---

<a id="q054"></a>

## #054 · 如何查询apt 安装的build-essential的路径

> 🕒 2026-09-29 21:18 ｜ DeepSeek 消息 ID 107–108 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

如何查询apt 安装的build-essential的路径

**回答**

#### 查询 `build-essential` 安装路径的方法

`build-essential` 是一个**元包（meta-package）**，它本身不包含文件，只是依赖其他包。所以查询它的路径需要分几个层面来看。

---

##### 方法1：查看包的依赖关系

```bash
apt-cache depends build-essential
```

输出示例：
```
build-essential
  依赖: libc6-dev
  依赖: gcc
  依赖: g++
  依赖: make
  依赖: dpkg-dev
```

这说明 `build-essential` 只是把 `gcc`、`make`、`libc6-dev` 等包捆绑在一起，本身没有实际文件。

---

##### 方法2：查看某个包安装的所有文件路径 ✅

```bash
# 查看 gcc 安装的文件
dpkg -L gcc

# 查看 make 安装的文件
dpkg -L make

# 查看 libc6-dev 安装的文件（标准C头文件在这里）
dpkg -L libc6-dev
```

输出示例（`dpkg -L gcc`）：
```
/usr/bin/gcc
/usr/bin/gcc-13
/usr/share/doc/gcc
...
```

---

##### 方法3：使用 `which` 和 `whereis` 查找可执行文件

```bash
which gcc
# 输出：/usr/bin/gcc

which make
# 输出：/usr/bin/make

whereis gcc
# 输出：gcc: /usr/bin/gcc /usr/lib/gcc /usr/share/man/man1/gcc.1.gz
```

---

##### 方法4：查找头文件路径

标准C头文件（如 `stdio.h`）由 `libc6-dev` 提供：

```bash
# 查找 stdio.h
dpkg -L libc6-dev | grep stdio.h

# 输出：/usr/include/stdio.h

# 或直接用 find
find /usr/include -name "stdio.h"
```

---

##### 方法5：查看 GCC 的搜索路径（最实用）✅

```bash
# 查看头文件搜索路径
echo | gcc -E -v -

# 查看库文件搜索路径
gcc -print-search-dirs

# 查看某个头文件的实际位置
echo '#include <stdio.h>' | gcc -E -v - 2>&1 | grep stdio.h
```

---

##### 方法6：查看包信息

```bash
# 查看包是否已安装
dpkg -l build-essential

# 查看包的详细信息
apt show build-essential

# 查看包的依赖
apt-cache depends build-essential
```

---

#### Ubuntu 上关键路径总结

| 内容 | 路径 |
|------|------|
| **可执行文件**（gcc, make, g++） | `/usr/bin/` |
| **标准C头文件**（stdio.h 等） | `/usr/include/` |
| **GCC 内部头文件** | `/usr/lib/gcc/x86_64-linux-gnu/13/include/` |
| **库文件**（.so, .a） | `/usr/lib/x86_64-linux-gnu/` |
| **GCC 内部库** | `/usr/lib/gcc/x86_64-linux-gnu/13/` |
| **文档** | `/usr/share/doc/` |
| **man 手册** | `/usr/share/man/` |

---

#### 完整验证示例

```bash
# 1. 查看 build-essential 依赖了哪些包
apt-cache depends build-essential

# 2. 查看 gcc 的安装文件
dpkg -L gcc | head -20

# 3. 查找 stdio.h 的位置
dpkg -L libc6-dev | grep "stdio.h"

# 4. 查看 gcc 实际使用的头文件搜索路径
echo | gcc -E -v - 2>&1 | sed -n '/#include <...> search starts here:/,/End of search list./p'

# 5. 查看 gcc 实际使用的库搜索路径
gcc -print-search-dirs

# 6. 查看 gcc 版本
gcc --version
```

---

#### 关键理解

`build-essential` 是**元包**，它的作用类似于一个"快捷方式"，让你一次性安装：
- `gcc` — C 编译器
- `g++` — C++ 编译器
- `make` — 构建工具
- `libc6-dev` — C 标准库开发文件（**包含 stdio.h 等头文件**）
- `dpkg-dev` — Debian 打包工具

所以当你问"build-essential 的路径"时，实际上是在问这些**子包**的路径。用 `dpkg -L <包名>` 就能查到任何一个包安装的所有文件。

<details>
<summary>💭 思考过程</summary>

*（耗时 1 秒）*

用户想知道如何查找通过 apt 安装的 build-essential 包的路径。让我提供一个全面的回答。

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q062"></a>

## #062 · 通过apt install build-essential的时候会不会安装gdb？

> 🕒 2026-09-30 20:21 ｜ DeepSeek 消息 ID 123–124 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

通过apt install build-essential的时候会不会安装gdb？

**回答**

`apt install build-essential` **不会**安装 `gdb`。

`build-essential` 是一个**元包（meta-package）**，它本身不包含任何文件，只是将编译 C/C++ 程序所必需的核心工具捆绑在一起[0]。

##### 📦 `build-essential` 实际包含的内容

根据 Ubuntu 官方软件包信息，它的依赖项（dep）明确列出了以下工具[\[1\]](https://packages.ubuntu.com/ja/noble/i386/build-essential)[\[2\]](https://packages.ubuntu.com/ja/focal-updates/riscv64/build-essential)：

*   **`dpkg-dev`**：Debian 包开发工具。
*   **`g++`**：GNU C++ 编译器。
*   **`gcc`**：GNU C 编译器。
*   **`libc6-dev`**：GNU C 标准库的开发文件（包含 `stdio.h` 等头文件）。
*   **`make`**：用于控制编译过程的构建工具。

你可以看到，这个列表里**并不包含 `gdb`**[\[3\]](http://www.yisu.com/ask/71666597.html)。`gdb`（GNU 调试器）在 Ubuntu 中是作为一个独立的软件包提供的。

##### 🛠️ 如何安装 GDB

如果你需要调试程序，需要**单独安装** `gdb`：

```bash
sudo apt install gdb
```

安装后，可以通过 `gdb --version` 来验证[\[5\]](http://www.yisu.com/ask/36323479.html)。

所以，如果你需要一个完整的、包含调试器的 C/C++ 开发环境，通常的安装命令是：

```bash
sudo apt install build-essential gdb
```

<details>
<summary>💭 思考过程</summary>

*（耗时 2 秒）*

用户想知道在Ubuntu上通过`apt install build-essential`安装时，是否会同时安装`gdb`。我需要提供准确的信息。为了全面获取信息，我将同时进行多项搜索，以覆盖`build-essential`的依赖关系、`gdb`的安装方式以及可能的官方文档说明。

*（耗时 1 秒）*

搜索结果显示了一些相关页面。为了全面了解`build-essential`的依赖关系以及它是否包含`gdb`，我们需要同时打开这些页面。其中，结果0、结果1、结果2、结果3、结果4、结果5、结果6都可能包含相关信息。

*（耗时 1 秒）*

根据已打开页面的信息，`build-essential`的依赖项包括`dpkg-dev`、`g++`、`gcc`、`libc6-dev`和`make`，并不包含`gdb`。多个来源（如结果0、结果1、结果2、结果3、结果4、结果5）都明确指出，需要单独安装`gdb`。因此，答案是否定的。我的回答将首先明确这一点，然后解释`build-essential`实际包含的内容，并说明如何安装`gdb`。

</details>

<details>
<summary>🔍 联网检索（4 条查询词 / 14 个来源）</summary>

查询词：`build-essential dependencies gdb`；`apt install build-essential gdb included`；`site:packages.ubuntu.com build-essential dependencies`；`build-essential package includes gdb`

主要来源：

- [The goal of this lab is two- fold](https://cs.pomona.edu/classes/cs181ca/labs/lab01.pdf#1#1)
- [https://git.flaxengine.com/Flax/FlaxEngine/commit/e90de5d815f4b5e45b8a9cb7987e922887c767a1…](https://git.flaxengine.com/Flax/FlaxEngine/commit/e90de5d815f4b5e45b8a9cb7987e922887c767a1.patch)
- [**Répertoires installés:** /usr/include/gc et /usr/share/doc/gc-8.2.12](http://fr.linuxfromscratch.org/archives/BLFS-13.0-systemd/BLFS-SYSD-BOOK-13.0-fr.pdf#277#97)
- [ubuntu怎么配c++编辑器](https://m.yisu.com/ask/48760942.html)
- [At present there does not seem to be a solution for pax on Rocky 9.](https://docs.rtems.org/docs/main/user.pdf#54#11)
- [Eclipse Community Forums: C / C++ IDE (CDT) » GDB in eclipse | The Eclipse Foundation](https://www.eclipse.org/forums/index.php/m/1831210/)
- [Debian下C++开发环境怎么搭](http://www.yisu.com/ask/71666597.html)
- [Ubuntu怎么安装C++环境](https://m.yisu.com/ask/57628775.html)
- [Devuan Wiki | Chomwitt / Metapackages](https://wiki.devuan.org/?n=Chomwitt.Metapackages?action=diff)
- [Revision 5655e7d9-6551-4cea-8bd8-3a7896663f83](https://unix.stackexchange.com/revisions/5655e7d9-6551-4cea-8bd8-3a7896663f83/view-source)
- [Ubuntu – noble の build-essential パッケージに関する詳細](https://packages.ubuntu.com/ja/noble/i386/build-essential)
- [パッケージ: build-essential (12.8ubuntu1.1) [ports]](https://packages.ubuntu.com/ja/focal-updates/riscv64/build-essential)
- [HOWTO install GNU toolchain](https://trisquel.info/en/wiki/howto-install-gnu-toolchain)
- [ubuntu c加加调试工具如何设](http://www.yisu.com/ask/36323479.html)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q072"></a>

## #072 · 我的Ubuntu中man命令不能用，怎么办？

> 🕒 2026-10-08 16:12 ｜ DeepSeek 消息 ID 143–144 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

我的Ubuntu中man命令不能用，怎么办？

**回答**

结论：Ubuntu 下 `man` 命令无法使用，通常是因为系统没有预装 `man-db` 这个包，执行一条安装命令就能解决[citation:7][citation:8]。

##### 🛠️ 解决方案：安装 man-db

请依次执行以下命令：

```bash
sudo apt update
sudo apt install man-db
```

安装完成后，运行 `man --version` 或 `man ls` 来验证是否恢复正常[citation:10][citation:13]。

##### 📄 如果安装后仍提示 “No manual entry for...”

这说明 `man` 程序已经装好，但缺少对应命令的**手册页文件**。你需要额外安装文档包[citation:16]：

```bash
# 安装核心手册页和开发手册页（适合C语言学习）
sudo apt install manpages manpages-dev manpages-posix manpages-posix-dev
```

##### 🔍 如果问题依旧，可以检查这两点

1.  **PATH 环境变量**：确保 `/usr/bin` 在你的 `PATH` 中。运行 `echo $PATH` 查看，若没有则执行 `export PATH=$PATH:/usr/bin`[citation:16]。
2.  **MANPATH 环境变量**：如果你手动设置过 `MANPATH`，它可能会覆盖默认的搜索路径。可以尝试 `unset MANPATH` 来清除它，然后重新测试[citation:16]。

<details>
<summary>🔍 联网检索（3 条查询词 / 10 个来源）</summary>

查询词：`Ubuntu 安装 man 命令`；`Ubuntu man: command not found`；`site:manpages.ubuntu.com man-db`

主要来源：

- [man man | Linux Command Library](https://linuxcommandlibrary.com/man/man)
- [[Bug] man コマンドが使用不能になる · Issue #2 · munepi/tlenv - Skip to content](https://github.com/munepi/tlenv/issues/2#1)
- [Ubuntu Manpage: manpath - man-db_2.13.1-1_amd64](https://manpages.ubuntu.com/manpages/resolute/en/man5/manpath.5.html#1)
- [中文linux man手册](https://cloud.tencent.cn/developer/information/%E4%B8%AD%E6%96%87linux%20man%E6%89%8B%E5%86%8C)
- [[elpa] externals/hyperbole 6b379c8e19 04/18: Install missing 'man' execu](https://mail.gnu.org/archive/html/emacs-elpa-diffs/2024-11/msg00702.html)
- [Ubuntu Manpage: NAVN - BESKRIVELSE](https://manpages.ubuntu.com/manpages/jammy/da/man5/manpath.5.html#1)
- [Why doesn't man work? | Linode Questions](https://www.linode.com/community/questions/17435/why-doesnt-man-work#answer-67579)
- [Fix - bash: man: command not found | Code2care](https://code2care.org/linux/fix-bash-man-command-not-found/#disqus_thread)
- [Linux基本指令总结（二） - Linux基本指令总结（二）](https://developer.aliyun.com/article/1514519#1)
- [Linux Ubuntu man文档的图文安装教程 - Linux Ubuntu man文档的图文安装教程](https://developer.aliyun.com/article/1350153#:~:text=man%E6%96%87%E6%A1%A3%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E6%8F%90%E4%BE%9B%E5%85%B3%E4%BA%8E%E7%89%B9%E5%AE%9A%E5%91%BD%E4%BB%A4%E3%80%81%E5%87%BD%E6%95%B0%E3%80%81%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E3%80%81%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%AD%89%E7%9A%84%E8%AF%A6%E7%BB%86%E8%AF%B4%E6%98%8E%E5%92%8C%E7%94%A8%E6%B3%95%E7%A4%BA%E4%BE%8B%E3%80%82,%E5%AE%83%E6%98%AF%E9%80%9A%E8%BF%87%E7%BB%88%E7%AB%AF%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%95%8C%E9%9D%A2%EF%BC%88CLI%EF%BC%89%E6%9F%A5%E7%9C%8B%E7%9A%84%EF%BC%8C%E7%94%A8%E6%88%B7%E5%8F%AF%E4%BB%A5%E6%A0%B9%E6%8D%AE%E9%9C%80%E8%A6%81%E6%B5%8F%E8%A7%88%E5%92%8C%E6%90%9C%E7%B4%A2man%E9%A1%B5%E6%9D%A5%E8%8E%B7%E5%8F%96%E7%9B%B8%E5%BA%94%E7%9A%84%E6%8A%80%E6%9C%AF%E4%BF%A1%E6%81%AF%E3%80%82%20man%E6%96%87%E6%A1%A3%E9%80%9A%E5%B8%B8%E5%8C%85%E5%90%AB%E5%91%BD%E4%BB%A4%E7%9A%84%E8%AF%AD%E6%B3%95%E3%80%81%E9%80%89%E9%A1%B9%E3%80%81%E7%A4%BA%E4%BE%8B%E5%92%8C%E5%85%B6%E4%BB%96%E7%9B%B8%E5%85%B3%E8%AF%B4%E6%98%8E%EF%BC%8C%E5%AF%B9%E4%BA%8E%E4%BD%BF%E7%94%A8%E5%92%8C%E7%90%86%E8%A7%A3%E5%91%BD%E4%BB%A4%E9%9D%9E%E5%B8%B8%E6%9C%89%E5%B8%AE%E5%8A%A9%E3%80%82#1)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q073"></a>

## #073 · $ man --version This system has been minimized by removing p…

> 🕒 2026-10-09 09:38 ｜ DeepSeek 消息 ID 145–146 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

```text
$ man --version
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, including manpages, you can run the 'unminimize'
command. You will still need to ensure the 'man-db' package is installed.
```

**回答**

你遇到的情况，是 Ubuntu 为了节省空间而采用的“最小化”安装策略导致的。`man` 命令本身并不缺，缺的是它依赖的文档和数据包。

好在系统已经给出了明确的解决方案，按照提示操作就可以恢复。

##### 🛠️ 恢复步骤

1.  **运行 `unminimize` 命令**：在终端中执行以下命令。它会重新安装被移除的文档和手册页，这个过程可能需要几分钟[citation:1][citation:2]。
    ```bash
    sudo unminimize
    ```

2.  **安装 `man-db`**：正如提示所说，`unminimize` 脚本可能不会自动安装 `man-db` 包[citation:4][citation:5][citation:8]。因此，在 `unminimize` 完成后，你还需要手动安装它[citation:1][citation:14]。
    ```bash
    sudo apt update
    sudo apt install man-db
    ```

##### 💡 如果 `unminimize` 失败怎么办？

在某些情况下，`unminimize` 可能会因为系统里有一些被“锁定”的包而执行失败[citation:15]。如果遇到这种情况，可以跳过 `unminimize`，直接安装 `man` 和 `man-db`：

```bash
sudo apt install man man-db
```

##### 💎 一条命令搞定

如果你不想分步操作，也可以把上面的步骤合并成一条命令来执行[citation:3]：

```bash
sudo apt update && sudo apt install -y man-db && yes | sudo unminimize
```

完成这些步骤后，再运行 `man ls` 之类的命令，就应该可以看到正常的手册页了。

<details>
<summary>🔍 联网检索（4 条查询词 / 10 个来源）</summary>

查询词：`How to restore man pages on a minimized Ubuntu system using the unminimize command`；`site:ubuntu.com unminimize`；`Ubuntu 的 unminimize 命令如何使用 恢复被最小化系统移除的手册页`；`Does the unminimize command on Ubuntu automatically install man-db`

主要来源：

- [Revision](https://superuser.com/revisions/1927277/1)
- [Ubuntu base and minimal images](https://ubuntu.com/cloud/public-cloud/docs/all-clouds-explanation/ubuntu-base-and-minimal-images/#1)
- [TIL: Unminimizing Ubuntu Docker images | Heitor's log](https://heitorpb.github.io/bla/unminimzing-docker-images/)
- [[Bug 1778777] Re: unminimize script is not restoring man-db - [Bug 1778777] Re: unminimize…](https://lists.ubuntu.com/archives/foundations-bugs/2018-July/361600.html#1)
- [[Bug 1778777] Re: unminimize script is not restoring man-db - [Bug 1778777] Re: unminimize…](https://lists.ubuntu.com/archives/foundations-bugs/2018-July/361601.html#1)
- [Ubuntu – 套件搜尋結果 -- unminimize](https://packages.ubuntu.com/search?lang=zh-tw&keywords=unminimize)
- [How to install man pages on an ubuntu docker image](https://stafura.dev/how-to-install-man-pages-on-an-ubuntu-docker-image/)
- [[Bug 1778777] [NEW] unminimize script is not restoring man-db](https://lists.ubuntu.com/archives/foundations-bugs/2018-June/360709.html)
- [软件包: unminimize (0.5build1) [ports]](https://packages.ubuntu.com/resolute/riscv64/unminimize)
- [Ubuntu unminimize操作指南](https://www.yisu.com/ask/54892654.html)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q074"></a>

## #074 · unminimize之后会把我之前安装的覆盖掉吗

> 🕒 2026-10-09 09:39 ｜ DeepSeek 消息 ID 147–148 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

unminimize之后会把我之前安装的覆盖掉吗

**回答**

**不会。** `unminimize` 只是把 Ubuntu 最小化安装时**刻意移除**的那些标准文档、手册页和辅助工具**补回来**，不会覆盖或删除你之前手动安装的任何软件。

可以把它理解成“把系统恢复到标准 Ubuntu 桌面/服务器该有的样子”，而不是“重装系统”。

---

#### 它具体会做什么？

`unminimize` 主要执行两件事：

1. **安装被移除的文档包**
   - 例如 `manpages`、`manpages-dev`、`info`、`docs` 等
   - 这些包本来就应该在标准 Ubuntu 里，只是被裁剪掉了

2. **重新安装被标记为“已移除”的软件包**
   - 它根据 `/var/lib/dpkg/status` 里的记录，把当初为了最小化而删掉的包重新装回来
   - 它**不会碰**你后来自己通过 `apt install` 装的包

---

#### 它不会做什么？

| 操作 | 是否会发生 |
|------|-----------|
| 删除你手动安装的软件 | ❌ 不会 |
| 覆盖你修改过的配置文件 | ❌ 不会（dpkg 遇到修改过的配置文件会询问） |
| 降级已安装的包 | ❌ 不会 |
| 重装整个系统 | ❌ 不会 |
| 清空 `/home` 或用户数据 | ❌ 不会 |

---

#### 唯一需要注意的地方

`unminimize` 会调用 `apt install` 安装一批包。如果某个包和你已装的包**有依赖冲突**，apt 可能会提示你是否要卸载/降级某些东西。这种情况极少，但如果出现，**仔细看提示再确认**即可。

例如可能出现：
```
The following packages will be REMOVED:
  some-package-you-installed
The following NEW packages will be installed:
  manpages manpages-dev ...
Do you want to continue? [Y/n]
```

如果看到要移除你重要的包，按 `n` 取消，然后手动安装你需要的部分：
```bash
sudo apt install man-db manpages manpages-dev
```

---

#### 更保守的做法

如果你完全不想冒险，可以**跳过 `unminimize`**，只装你需要的：

```bash
sudo apt update
sudo apt install man-db manpages manpages-dev
```

这样只安装 `man` 手册和开发手册，不会触发其他包的变动。对于 C 语言学习，这些已经足够。

---

#### 总结

| 问题 | 答案 |
|------|------|
| `unminimize` 会覆盖我装的软件吗？ | ❌ 不会 |
| 会删除我的数据吗？ | ❌ 不会 |
| 会改我的配置文件吗？ | 基本不会，如有冲突会询问 |
| 最安全的方式？ | 只装 `man-db manpages manpages-dev` |

**放心执行 `unminimize`**，它只是把系统补全，不会破坏你已有的环境。如果实在不放心，就用保守方案，只装 `man` 相关的几个包。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q075"></a>

## #075 · 编译时-lssl报错，没有libssl.a的包

> 🕒 2026-10-09 09:52 ｜ DeepSeek 消息 ID 149–150 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

编译时-lssl报错，没有libssl.a的包

**回答**

`-lssl` 报错，核心原因通常是**只装了运行时库，没装开发库**，或者**链接顺序不对**。

##### 🔍 先判断是哪种情况

| 报错信息 | 原因 | 解决方法 |
|---|---|---|
| `cannot find -lssl` | 系统里根本没有 `libssl.so` 或 `libssl.a` | 安装 `libssl-dev` |
| `undefined reference to SSL_xxx` | 库文件存在，但链接器没找到符号 | 检查 `-lssl -lcrypto` 的顺序 |

##### 📦 安装开发库

Ubuntu 下 OpenSSL 的开发包（包含头文件和静态/动态库）叫 **`libssl-dev`**[citation:1][citation:2][citation:3]。

```bash
sudo apt update
sudo apt install libssl-dev
```

装完后，库文件会出现在 `/usr/lib/x86_64-linux-gnu/` 下，你会看到 `libssl.so` 和 `libssl.a`[citation:2][citation:5]。

##### ⚠️ 关于静态库 `libssl.a`

`libssl-dev` 包**确实会提供 `libssl.a`**（静态库）[citation:2][citation:17]。但 Ubuntu 的一些版本里，OpenSSL 3.x 的静态库可能和动态库**不在同一个包里**。如果 `apt install libssl-dev` 之后还是找不到 `libssl.a`，可以再装：

```bash
sudo apt install libssl3-dev
```

或者直接确认文件是否存在：

```bash
find /usr/lib -name "libssl.a" 2>/dev/null
```

##### 🔗 链接顺序很关键

即使库装好了，`-lssl` 和 `-lcrypto` 的**顺序也不能错**。因为 `libssl` 依赖 `libcrypto`，链接器是从左到右解析的：

```makefile
LDLIBS = -lssl -lcrypto    # ✅ 正确
LDLIBS = -lcrypto -lssl    # ❌ 可能报 undefined reference
```

##### 🛠️ 回到你的 Makefile

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread -lssl -lcrypto

helloworld: helloworld.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
```

##### 💡 补充：如果 `libssl-dev` 安装后还是报错

有一个常见坑是 **`pkg-config` 没装**，导致编译器找不到 OpenSSL 的编译参数[citation:7][citation:12]。补上：

```bash
sudo apt install pkg-config
```

然后验证：

```bash
pkg-config --libs openssl
# 应该输出：-lssl -lcrypto
```

##### 📝 总结

1. **装开发包**：`sudo apt install libssl-dev`
2. **确认静态库**：`find /usr/lib -name "libssl.a"`，没有就补 `libssl3-dev`
3. **检查链接顺序**：`-lssl` 必须在 `-lcrypto` 前面
4. **补 `pkg-config`**：如果装完还是找不到

<details>
<summary>🔍 联网检索（4 条查询词 / 10 个来源）</summary>

查询词：`Ubuntu 编译报错 "cannot find -lssl" 没有 libssl.a 怎么解决`；`Ubuntu 如何安装 libssl-dev 静态库 libssl.a`；`apt-get install libssl-dev`；`Ubuntu install libssl-dev static library libssl.a`

主要来源：

- [Revision 8bb7748e-1095-45e4-a45f-fb6c46300416](https://stackoverflow.com/revisions/8bb7748e-1095-45e4-a45f-fb6c46300416/view-source)
- [Ubuntu 20上安装使用OpenSSL](https://cloud.baidu.com/article/2975359)
- [页面历史记录](https://wiki.rdkcentral.com/pages/diffpagesbyversion.action?pageId=135856543&originalVersion=4&revisedVersion=7)
- [Revision e84ab7d4-63cd-4076-ade2-a2b8f3f43351](https://stackoverflow.com/revisions/e84ab7d4-63cd-4076-ade2-a2b8f3f43351/view-source)
- [ubuntu c++如何静态链接](http://m.yisu.com/ask/4832773.html)
- [[openssl](https://github](https://raw.githubusercontent.com/emqx/neuron/main/Install-dependencies.md#1)
- [Revision 59aa53c8-4527-4ab6-985e-bc1bcba28b8a](https://stackoverflow.com/revisions/59aa53c8-4527-4ab6-985e-bc1bcba28b8a/view-source)
- [ubuntu c++依赖库怎么安装](http://www.yisu.com/ask/58234029.html)
- [Nginx 源码安装怎么解决 openssl 依赖库报错 - Nginx 源码安装怎么解决 openssl 依赖库报错](https://www.php.cn/faq/2868208.html#1)
- [ubuntu gcc为何报错找不到库](http://www.yisu.com/ask/73059550.html)

</details>

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q076"></a>

## #076 · 怎样通过apt检查是否安装了libssl-dev

> 🕒 2026-10-09 10:00 ｜ DeepSeek 消息 ID 151–152 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

怎样通过apt检查是否安装了libssl-dev

**回答**

#### 方法1：`dpkg -l`（最常用）

```bash
dpkg -l libssl-dev
```

- **已安装**：输出中以 `ii` 开头
  ```
  Desired=Unknown/Install/Remove/Purge/Hold
  | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
  |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
  ||/ Name           Version      Architecture Description
  +++-==============-============-============-=================================
  ii  libssl-dev:amd64 3.0.2-0ubuntu1 amd64     Secure Sockets Layer toolkit
  ```
- **未安装**：输出 `dpkg-query: no packages found matching libssl-dev`

---

#### 方法2：`apt list`（更直观）

```bash
apt list --installed | grep libssl-dev
```

- **已安装**：输出 `libssl-dev/jammy-updates,now 3.0.2-0ubuntu1 amd64 [installed]`
- **未安装**：无输出

---

#### 方法3：`dpkg -s`（查看详细信息）

```bash
dpkg -s libssl-dev
```

- **已安装**：输出包的详细信息（版本、依赖、描述等）
- **未安装**：输出 `dpkg-query: package 'libssl-dev' is not installed`

---

#### 方法4：`apt-cache policy`（查看安装状态和可用版本）

```bash
apt-cache policy libssl-dev
```

输出示例：
```
libssl-dev:
  Installed: 3.0.2-0ubuntu1
  Candidate: 3.0.2-0ubuntu1
  Version table:
 *** 3.0.2-0ubuntu1 500
        500 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
        100 /var/lib/dpkg/status
```

- **已安装**：`Installed:` 后面有版本号
- **未安装**：`Installed: (none)`

---

#### 方法5：检查关键文件是否存在（最直接）

```bash
# 检查头文件
ls /usr/include/openssl/ssl.h

# 检查动态库
ls /usr/lib/x86_64-linux-gnu/libssl.so

# 检查静态库
ls /usr/lib/x86_64-linux-gnu/libssl.a
```

如果 `ssl.h` 和 `libssl.so` 都存在，基本可以确认 `libssl-dev` 已安装。

---

#### 推荐组合

```bash
# 一行命令快速判断
dpkg -l libssl-dev 2>/dev/null | grep "^ii" && echo "已安装" || echo "未安装"
```

或者：

```bash
apt list --installed 2>/dev/null | grep -q libssl-dev && echo "已安装" || echo "未安装"
```

---

#### 总结

| 命令 | 用途 | 已安装标志 |
|------|------|-----------|
| `dpkg -l libssl-dev` | 最常用 | 行首 `ii` |
| `apt list --installed \| grep libssl-dev` | 直观 | `[installed]` |
| `dpkg -s libssl-dev` | 详细信息 | 输出包信息 |
| `apt-cache policy libssl-dev` | 版本信息 | `Installed:` 有版本号 |
| `ls /usr/include/openssl/ssl.h` | 最直接 | 文件存在 |

**最推荐**：`dpkg -l libssl-dev | grep "^ii"`，简单可靠。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q077"></a>

## #077 · 如何查看安装包的development libraries, header files, and manpages

> 🕒 2026-10-09 10:06 ｜ DeepSeek 消息 ID 153–154 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

如何查看安装包的development libraries, header files, and manpages

**回答**

在 Ubuntu/Debian 上，查看一个包是否包含开发库、头文件和手册页，可以用以下几种方法。

---

#### 一、查看已安装包的文件列表

##### 方法1：`dpkg -L`（最常用）✅

```bash
dpkg -L libssl-dev
```

这会列出 `libssl-dev` 安装的所有文件。你可以配合 `grep` 分类查看：

```bash
# 查看头文件（.h）
dpkg -L libssl-dev | grep "\.h$"

# 查看动态库（.so）
dpkg -L libssl-dev | grep "\.so$"

# 查看静态库（.a）
dpkg -L libssl-dev | grep "\.a$"

# 查看手册页（man）
dpkg -L libssl-dev | grep "man"
```

**示例输出**：
```
/usr/include/openssl/ssl.h          ← 头文件
/usr/lib/x86_64-linux-gnu/libssl.so ← 动态库
/usr/lib/x86_64-linux-gnu/libssl.a  ← 静态库
/usr/share/man/man3/SSL_...         ← 手册页
```

---

##### 方法2：`dpkg -S`（反查文件属于哪个包）

如果你已经知道文件路径，想知道它属于哪个包：

```bash
# 查看 ssl.h 属于哪个包
dpkg -S /usr/include/openssl/ssl.h
# 输出：libssl-dev:amd64: /usr/include/openssl/ssl.h

# 查看 libssl.so 属于哪个包
dpkg -S /usr/lib/x86_64-linux-gnu/libssl.so
```

---

#### 二、查看未安装包的文件列表

如果包还没装，可以用 `apt-file` 查看它包含哪些文件。

##### 安装 `apt-file`

```bash
sudo apt update
sudo apt install apt-file
sudo apt-file update
```

##### 使用 `apt-file`

```bash
# 查看 libssl-dev 包含的所有文件
apt-file list libssl-dev

# 查找某个头文件属于哪个包
apt-file search openssl/ssl.h

# 查找某个库属于哪个包
apt-file search libssl.so
```

---

#### 三、按类别查看开发文件

##### 头文件（Header Files）

```bash
# 方法1：dpkg -L + grep
dpkg -L libssl-dev | grep "/usr/include"

# 方法2：直接看目录
ls /usr/include/openssl/
```

##### 开发库（Development Libraries）

```bash
# 动态库（.so）
dpkg -L libssl-dev | grep "\.so"

# 静态库（.a）
dpkg -L libssl-dev | grep "\.a"

# 或者直接看库目录
ls /usr/lib/x86_64-linux-gnu/libssl*
```

##### 手册页（Manpages）

```bash
# 查看包的手册页
dpkg -L libssl-dev | grep "man"

# 直接查看某个函数的手册
man SSL_new
man 3 SSL_CTX_new
```

---

#### 四、一次性查看包的所有信息

##### `apt show`（包描述）

```bash
apt show libssl-dev
```

输出包含版本、依赖、描述等，但不包含文件列表。

##### `dpkg -s`（已安装包的详细信息）

```bash
dpkg -s libssl-dev
```

##### 组合命令：分类查看

```bash
PKG=libssl-dev

echo "=== 头文件 ==="
dpkg -L $PKG | grep "\.h$"

echo "=== 动态库 ==="
dpkg -L $PKG | grep "\.so"

echo "=== 静态库 ==="
dpkg -L $PKG | grep "\.a$"

echo "=== 手册页 ==="
dpkg -L $PKG | grep "man"

echo "=== 其他 ==="
dpkg -L $PKG | grep -v "\.h$\|\.so\|\.a$\|man"
```

---

#### 五、常见开发包的命名规律

Ubuntu 里开发包通常以 `-dev` 结尾：

| 运行时包 | 开发包 | 提供内容 |
|---------|--------|---------|
| `libssl3` | `libssl-dev` | 头文件 + .so + .a + man |
| `libcurl4` | `libcurl4-openssl-dev` | 头文件 + 库 |
| `zlib1g` | `zlib1g-dev` | 头文件 + 库 |
| `libsqlite3-0` | `libsqlite3-dev` | 头文件 + 库 |
| `libpthread` | （包含在 `libc6-dev` 中） | 头文件 |

**规律**：
- **运行时包**：只含 `.so` 动态库（供程序运行）
- **开发包**：含 `.h` 头文件 + `.so` 链接符号 + 可选 `.a` 静态库 + man 手册

---

#### 六、验证开发环境是否完整

以 OpenSSL 为例，检查三要素：

```bash
# 1. 头文件
ls /usr/include/openssl/ssl.h
# 或
dpkg -L libssl-dev | grep "openssl/ssl.h"

# 2. 库文件
ls /usr/lib/x86_64-linux-gnu/libssl.so
ls /usr/lib/x86_64-linux-gnu/libssl.a

# 3. 手册页
man SSL_new
# 或
dpkg -L libssl-dev | grep "man3"
```

如果三者都存在，说明开发环境完整。

---

#### 七、总结

| 需求 | 命令 |
|------|------|
| 查看已安装包的文件列表 | `dpkg -L <包名>` |
| 查看头文件 | `dpkg -L <包名> \| grep "\.h$"` |
| 查看动态库 | `dpkg -L <包名> \| grep "\.so"` |
| 查看静态库 | `dpkg -L <包名> \| grep "\.a$"` |
| 查看手册页 | `dpkg -L <包名> \| grep "man"` |
| 反查文件属于哪个包 | `dpkg -S <文件路径>` |
| 查看未安装包的文件列表 | `apt-file list <包名>` |
| 查找某个文件属于哪个包 | `apt-file search <文件名>` |
| 查看包描述 | `apt show <包名>` |

**最常用组合**：
```bash
# 查看某个开发包提供的所有文件
dpkg -L libssl-dev

# 只看头文件和库
dpkg -L libssl-dev | grep -E "\.(h|so|a)$"
```

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q128"></a>

## #128 · 在Ubuntu Linux上如何查机器物理内存的大小？

> 🕒 2026-10-16 12:04 ｜ DeepSeek 消息 ID 255–256 ｜ 主题：Ubuntu 系统与软件包管理（man / apt / unminimize）

**提问**

在Ubuntu Linux上如何查机器物理内存的大小？

**回答**

在 Ubuntu Linux 上查看物理内存大小，有下面几种常用方法。

---

#### 一、`free -h`（最常用）✅

```bash
free -h
```

输出示例：
```
              total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.1Gi       4.5Gi       100Mi       1.1Gi       5.2Gi
Swap:          2.0Gi          0B       2.0Gi
```

**说明**：
| 字段 | 含义 |
|------|------|
| `total` | **物理内存总量** |
| `used` | 已用内存 |
| `free` | 空闲内存 |
| `shared` | 共享内存 |
| `buff/cache` | 缓存 |
| `available` | 可用内存 |

**注意**：`7.7Gi` 而不是 `8Gi`，因为：
- 硬件保留了一部分内存
- 内核占用了一部分
- 所以可用总量略小于标称值

---

#### 二、`/proc/meminfo`（最详细）✅

```bash
cat /proc/meminfo
```

输出示例：
```
MemTotal:        8107604 kB
MemFree:         4721424 kB
MemAvailable:    5489360 kB
Buffers:          123456 kB
Cached:          1234567 kB
...
```

**关键字段**：
| 字段 | 含义 |
|------|------|
| `MemTotal` | **物理内存总量**（KB） |
| `MemFree` | 空闲内存 |
| `MemAvailable` | 可用内存（估算） |
| `Buffers` | 块设备缓存 |
| `Cached` | 文件缓存 |

**只看总量**：
```bash
grep MemTotal /proc/meminfo
```

输出：
```
MemTotal:        8107604 kB
```

**换算**：
```
8107604 kB ≈ 7.73 GB
```

---

#### 三、`lshw`（硬件信息）✅

```bash
sudo lshw -short -C memory
```

输出示例：
```
H/W path        Device     Class          Description
=====================================================
/0/0                       memory         128KiB BIOS
/0/4/0                     memory         8GiB System Memory
/0/4/0/0                   memory         8GiB SODIMM DDR4 Synchronous 3200 MHz
```

**更详细**：
```bash
sudo lshw -class memory
```

---

#### 四、`dmidecode`（DMI 信息）✅

```bash
sudo dmidecode -t memory
```

输出示例：
```
Handle 0x0008, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x0007
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 8192 MB
        Form Factor: SODIMM
        Set: None
        Locator: ChannelA-DIMM0
        Bank Locator: BANK 0
        Type: DDR4
        Speed: 3200 MT/s
        Manufacturer: Samsung
        ...
```

**只看总量**：
```bash
sudo dmidecode -t memory | grep "Size:"
```

输出：
```
        Size: 8192 MB
        Size: No Module Installed
```

**说明**：每根内存条一行，`No Module Installed` 表示空闲插槽。

---

#### 五、`lsmem`（专用工具）✅

```bash
lsmem
```

输出示例：
```
RANGE                                 SIZE  STATE REMOVABLE BLOCK
0x0000000000000000-0x000000007fffffff   2G online        yes  0-15
0x0000000100000000-0x000000017fffffff   2G online        yes 16-31
0x0000000200000000-0x000000027fffffff   2G online        yes 32-47
0x0000000300000000-0x000000037fffffff   2G online        yes 48-63

Memory block size:       128M
Total online memory:       8G
Total offline memory:      0B
```

**如果没有 `lsmem`**：
```bash
sudo apt install util-linux
```

---

#### 六、`top` 或 `htop`（实时监控）

##### `top`
```bash
top
```

输出顶部：
```
top - 10:30:00 up 1 day,  2:15,  1 user,  load average: 0.52, 0.58, 0.59
Tasks: 123 total,   1 running, 122 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.0 sy,  0.0 ni, 93.5 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7917.6 total,   4610.4 free,   1234.5 used,   2072.7 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5360.2 avail Mem
```

##### `htop`（更直观）
```bash
sudo apt install htop
htop
```

顶部显示内存条：
```
  0[|||||||||            50.0%]   Tasks: 123, 456 thr; 1 running
  1[||||||               30.0%]   Load average: 0.52 0.58 0.59
  Mem[|||||||||||||  2.1G/7.7G]   Uptime: 1 day, 2:15:00
  Swp[                0K/2.0G]
```

---

#### 七、`/proc/meminfo` 各字段详解

```bash
cat /proc/meminfo | head -20
```

输出示例：
```
MemTotal:        8107604 kB    ← 物理内存总量
MemFree:         4721424 kB    ← 完全空闲的内存
MemAvailable:    5489360 kB    ← 可用内存（估算，含可回收缓存）
Buffers:           32768 kB    ← 块设备缓存
Cached:          2345678 kB    ← 文件缓存
SwapCached:            0 kB    ← 交换缓存
Active:          1234567 kB    ← 活跃内存
Inactive:         987654 kB    ← 非活跃内存
...
SwapTotal:       2097148 kB    ← 交换空间总量
SwapFree:        2097148 kB    ← 空闲交换空间
```

**关键字段**：
| 字段 | 含义 |
|------|------|
| `MemTotal` | 物理内存总量 |
| `MemFree` | 完全空闲 |
| `MemAvailable` | 可用（推荐参考） |
| `SwapTotal` | 交换空间总量 |

---

#### 八、一行命令快速查看

```bash
# 方法1：free -h
free -h | awk '/^Mem:/{print $2}'

# 方法2：/proc/meminfo
grep MemTotal /proc/meminfo | awk '{print $2/1024/1024 " GB"}'

# 方法3：dmidecode
sudo dmidecode -t memory | awk '/Size:/ && !/No Module/{sum += $2} END {print sum/1024 " GB"}'
```

---

#### 九、各方法对比

| 方法 | 优点 | 缺点 | 是否需要 root |
|------|------|------|--------------|
| `free -h` | 简单、快速 | 略小于实际 | ❌ |
| `/proc/meminfo` | 详细、实时 | 需要解析 | ❌ |
| `lshw` | 硬件信息全 | 需安装、需 root | ✅ |
| `dmidecode` | 硬件信息最全 | 需 root、需安装 | ✅ |
| `lsmem` | 直观显示内存块 | 需安装 | ❌ |
| `top` | 实时监控 | 输出多 | ❌ |
| `htop` | 图形化 | 需安装 | ❌ |

---

#### 十、总结

| 需求 | 推荐命令 |
|------|---------|
| **快速查看总量** | `free -h` |
| **详细内存信息** | `cat /proc/meminfo` |
| **硬件信息（内存条数量、型号）** | `sudo dmidecode -t memory` |
| **内存块布局** | `lsmem` |
| **实时监控** | `top` 或 `htop` |
| **一行命令** | `grep MemTotal /proc/meminfo` |

**最常用**：
```bash
free -h
```

**最详细**：
```bash
cat /proc/meminfo
```

**硬件层面**：
```bash
sudo dmidecode -t memory | grep "Size:"
```

**注意**：
- `free -h` 显示的是**操作系统可用内存**，略小于物理内存（因为硬件保留和内核占用）
- `dmidecode` 显示的是**物理内存条**的容量，是硬件真实容量
- 例如 `free` 显示 `7.7Gi`，`dmidecode` 显示 `8192 MB`（8 GB），两者都对

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---
