<a id="top"></a>

# Makefile 与项目目录结构

> 归档自 DeepSeek 分享对话：<https://chat.deepseek.com/share/4fgpffcy16vh4nzis8>  
> 本主题共 **6 轮**问答（原始全局编号 #069–#196），时间 2026-10-07 ~ 2026-10-26。  
> 提问与回答均为原始内容；回答内的小标题层级整体下沉 2 级以适配本文件结构，`[reference:N]` 引用占位符已尽量还原为来源链接。  
> [« 返回总索引](./README.md)

## 本主题目录

- [#069 如果在编译的时候需要链接math库，在Makefile中怎么加呢](#q069)
- [#070 Makefile是这样：CC = gcc CFLAGS = -std=c99 -pedantic-errors -Wal…](#q070)
- [#149 如果我想把编译出来的文件放到./bin目录中，Makefile中如何改%: %.c $(CC) $(CFLAGS) -o…](#q149)
- [#194 刚才的Makefile文件改一下，编译结果都放在bin/目录中](#q194)
- [#195 可执行文件在bin/目录，中间文件.o在obj/目录](#q195)
- [#196 常用的是将.c文件放到src/目录，*.h文件放到include目录吗](#q196)

---

<a id="q069"></a>

## #069 · 如果在编译的时候需要链接math库，在Makefile中怎么加呢

> 🕒 2026-10-07 16:40 ｜ DeepSeek 消息 ID 137–138 ｜ 主题：Makefile 与项目目录结构

**提问**

如果在编译的时候需要链接math库，在Makefile中怎么加呢

**回答**

在 Makefile 中链接数学库（`libm`），需要加上 **`-lm`** 选项。关键是要把它放在**链接命令的末尾**，因为链接器对库的顺序有要求。

---

#### 一、最简单的方式

```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -g
LDLIBS = -lm

target: target.c
	$(CC) $(CFLAGS) -o target target.c $(LDLIBS)
```

或者直接写在命令里：

```makefile
target: target.c
	$(CC) $(CFLAGS) -o target target.c -lm
```

---

#### 二、为什么 `-lm` 必须放在最后？

链接器解析符号时是**从左到右**扫描的：

```
gcc -o target target.c -lm     # ✅ 正确
gcc -o target -lm target.c     # ⚠️ 可能失败
```

- 如果 `-lm` 放在 `target.c` **前面**，链接器先看到 `-lm`，但此时还不知道 `target.c` 里需要 `sin`、`cos` 等符号，所以 `libm` 里的符号不会被引用。
- 如果 `-lm` 放在 `target.c` **后面**，链接器已经知道需要 `sin`、`cos`，就会去 `libm` 里找。

---

#### 三、完整的 Makefile 模板

```makefile
# 编译器
CC = gcc

# 编译选项
CFLAGS = -std=c99 -Wall -Wextra -g -O0

# 链接选项（库）
LDLIBS = -lm

# 目标文件
TARGET = my_program

# 源文件
SRCS = main.c utils.c

# 目标文件（.o）
OBJS = $(SRCS:.c=.o)

# 默认目标
all: $(TARGET)

# 链接
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^ $(LDLIBS)

# 编译每个 .c 为 .o
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 清理
clean:
	rm -f $(OBJS) $(TARGET)

# 运行
run: $(TARGET)
	./$(TARGET)

.PHONY: all clean run
```

**关键点**：
- `$@` = 目标文件名（`my_program`）
- `$^` = 所有依赖文件（`main.o utils.o`）
- `$<` = 第一个依赖文件（`main.c`）
- `$(LDLIBS)` 放在**最后**

---

#### 四、多个库的情况

如果需要链接多个库，顺序也很重要：

```makefile
LDLIBS = -lm -lpthread -lssl -lcrypto
```

**规则**：
- 被依赖的库放在**后面**
- 例如 `libssl` 依赖 `libcrypto`，所以 `-lssl` 在前，`-lcrypto` 在后

```makefile
LDLIBS = -lssl -lcrypto -lm
```

---

#### 五、区分 `CFLAGS`、`LDFLAGS`、`LDLIBS`

| 变量 | 用途 | 示例 |
|------|------|------|
| `CFLAGS` | 编译选项 | `-std=c99 -Wall -g -O2` |
| `CPPFLAGS` | 预处理选项（头文件路径） | `-I/usr/local/include` |
| `LDFLAGS` | 链接器选项（库路径） | `-L/usr/local/lib` |
| `LDLIBS` | 要链接的库 | `-lm -lpthread` |

**完整示例**：

```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -g -O2
CPPFLAGS = -I/usr/local/include
LDFLAGS = -L/usr/local/lib
LDLIBS = -lm -lpthread

TARGET = my_program
SRCS = main.c
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)

%.o: %.c
	$(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

编译命令展开后：
```bash
gcc -std=c99 -Wall -g -O2 -c main.c -o main.o -I/usr/local/include
gcc -std=c99 -Wall -g -O2 -L/usr/local/lib -o my_program main.o -lm -lpthread
```

---

#### 六、常见错误

##### 错误1：`-lm` 放在前面

```makefile
# ❌ 错误
$(CC) -lm -o target target.c
```

**报错**：
```
undefined reference to `sin'
undefined reference to `cos'
```

##### 错误2：忘记 `-lm`

```makefile
# ❌ 错误
$(CC) -o target target.c
```

**报错**：
```
undefined reference to `sqrt'
```

##### 错误3：`-lm` 写成 `-l m`

```makefile
# ❌ 错误
$(CC) -o target target.c -l m
```

`-l` 和库名之间**不能有空格**。

---

#### 七、验证数学库是否生效

写一个测试程序：

```c
// test_math.c
#include <stdio.h>
#include <math.h>

int main() {
    double x = 2.0;
    printf("sqrt(2) = %f\n", sqrt(x));
    printf("sin(0) = %f\n", sin(0));
    printf("pow(2, 10) = %f\n", pow(2, 10));
    return 0;
}
```

Makefile：
```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -g
LDLIBS = -lm

test_math: test_math.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)

clean:
	rm -f test_math
```

编译运行：
```bash
make
./test_math
```

输出：
```
sqrt(2) = 1.414214
sin(0) = 0.000000
pow(2, 10) = 1024.000000
```

---

#### 八、总结

| 要点 | 说明 |
|------|------|
| 链接数学库 | 加 `-lm` |
| 位置 | 必须放在**源文件或目标文件之后** |
| 推荐变量 | 放在 `LDLIBS` 中 |
| 多库顺序 | 被依赖的库放在后面 |
| 常见错误 | 忘记 `-lm`、顺序错误、`-l m` 有空格 |

**最小可用模板**：

```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -g
LDLIBS = -lm

target: target.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)

clean:
	rm -f target
```

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q070"></a>

## #070 · Makefile是这样：CC = gcc CFLAGS = -std=c99 -pedantic-errors -Wal…

> 🕒 2026-10-08 10:11 ｜ DeepSeek 消息 ID 139–140 ｜ 主题：Makefile 与项目目录结构

**提问**

```text
Makefile是这样：CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread

SRCS = helloworld.c para_pass.c

helloworld: helloworld.c
	$(CC) $(CFLAGS) -o helloworld helloworld.c $(LDLIBS)

para_pass: para_pass.c
	$(CC) $(CFLAGS) -o para_pass para_pass.c $(LDLIBS)

clean:
	rm -f helloworld
	rm -f para_pass。但是make的时候只编译了helloworld.c
```

**回答**

#### 问题原因

你的 Makefile 中，**默认目标（第一个目标）是 `helloworld`**。执行 `make` 时，如果只写 `make` 不带参数，Make 只会构建**第一个目标**及其依赖，也就是 `helloworld`。

`para_pass` 虽然是独立目标，但**不会被自动构建**，因为它不是 `helloworld` 的依赖。

---

#### 验证

```bash
make          # 只编译 helloworld
make para_pass # 单独编译 para_pass
make clean    # 清理两个
```

---

#### 解决方案

##### 方案1：添加 `all` 目标（推荐）✅

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread

SRCS = helloworld.c para_pass.c

# 默认目标：构建所有可执行文件
all: helloworld para_pass

helloworld: helloworld.c
	$(CC) $(CFLAGS) -o helloworld helloworld.c $(LDLIBS)

para_pass: para_pass.c
	$(CC) $(CFLAGS) -o para_pass para_pass.c $(LDLIBS)

clean:
	rm -f helloworld para_pass

.PHONY: all clean
```

**执行**：
```bash
make          # 构建所有目标
make all      # 同上
make helloworld  # 只构建 helloworld
make para_pass   # 只构建 para_pass
make clean    # 清理
```

**关键点**：
- `all` 是**约定俗成的默认目标名**，放在最前面
- `.PHONY: all clean` 声明 `all` 和 `clean` 是"伪目标"，不是实际文件

---

##### 方案2：使用模式规则 + 变量（更简洁）✅

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread

# 所有可执行文件名 = 去掉 .c 后缀的源文件名
TARGETS = helloworld para_pass

all: $(TARGETS)

# 模式规则：所有可执行文件都由同名 .c 文件生成
%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)

clean:
	rm -f $(TARGETS)

.PHONY: all clean
```

**优点**：
- 新增源文件只需改 `TARGETS` 一行
- `%: %.c` 是模式规则，自动匹配 `helloworld.c → helloworld`
- `$@` = 目标名，`$<` = 第一个依赖（源文件）

---

##### 方案3：更完整的通用模板 ✅

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread

# 自动找出当前目录下所有 .c 文件对应的可执行文件名
TARGETS = $(patsubst %.c,%,$(wildcard *.c))

all: $(TARGETS)

%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)

clean:
	rm -f $(TARGETS)

.PHONY: all clean
```

**说明**：
- `wildcard *.c` 找出所有 `.c` 文件
- `patsubst %.c,%,...` 把 `.c` 后缀去掉，得到可执行文件名列表
- 新增 `.c` 文件后**不用改 Makefile**

**注意**：如果项目里有**多个 `.c` 需要链接成一个程序**，这个模板不适用（它假设每个 `.c` 都是独立程序）。

---

#### 方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| 方案1：显式 `all` | 直观、易理解 | 新增文件要改 Makefile | 少量固定目标 |
| 方案2：模式规则 | 简洁、自动匹配 | 需要手动列目标 | 多个独立程序 |
| 方案3：自动查找 | 全自动 | 假设每个 `.c` 独立 | 批量独立程序 |

---

#### 推荐

对于你的教学场景（多个独立小程序），**方案2** 最合适：

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
LDLIBS = -lm -lpthread

TARGETS = helloworld para_pass

all: $(TARGETS)

%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)

clean:
	rm -f $(TARGETS)

.PHONY: all clean
```

**使用**：
```bash
make             # 编译所有
make para_pass   # 只编译 para_pass
make clean       # 清理
```

---

#### 补充：为什么 `make` 默认只构建第一个目标？

Make 的设计哲学是：**默认构建"第一个目标"**，因为传统上 Makefile 的第一个目标通常是最终产物。这给了灵活性——你可以把辅助目标放在前面，但要注意用 `.DEFAULT_GOAL` 或把 `all` 放在最前面。

```makefile
# 显式指定默认目标（GNU Make 3.81+）
.DEFAULT_GOAL := all
```

但更常见的做法就是**把 `all` 放在最前面**。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q149"></a>

## #149 · 如果我想把编译出来的文件放到./bin目录中，Makefile中如何改%: %.c $(CC) $(CFLAGS) -o…

> 🕒 2026-10-20 14:15 ｜ DeepSeek 消息 ID 297–298 ｜ 主题：Makefile 与项目目录结构

**提问**

```text
如果我想把编译出来的文件放到./bin目录中，Makefile中如何改%: %.c
        $(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
        @echo "Built $@"
```

**回答**

要把编译产物统一放到 `./bin` 目录下，你需要修改 Makefile 里的**目标路径**和**编译命令的输出路径**。

直接改成下面这样即可：

```makefile
# 1. 创建 bin 目录（确保目录存在）
$(shell mkdir -p bin)

# 2. 修改模式规则，让目标变成 bin/%
bin/%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
	@echo "Built $@"
```

##### 为什么这么改？

-   **`bin/%: %.c`**：这样 Make 会去寻找 `xx.c` 来生成 `bin/xx`。由于 `$@` 代表目标（即 `bin/xx`），`$<` 代表源文件（即 `xx.c`），所以 `-o $@` 就会自动把输出放到 `bin/` 目录下。
-   **`$(shell mkdir -p bin)`**：`-o bin/xx` 要求 `bin` 目录必须存在，否则会报错。这句利用 Make 的 `shell` 函数在解析阶段自动创建该目录（`-p` 保证目录已存在时不会报错）。

---

##### 💡 如果你使用的是之前提到的 `all` 和 `TARGETS` 变量

如果你之前的 Makefile 是这样的：

```makefile
TARGETS = helloworld para_pass
all: $(TARGETS)

%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
```

那么你需要把 `TARGETS` 的路径也加上 `bin/`，否则它会在当前目录找目标：

```makefile
TARGETS = bin/helloworld bin/para_pass

all: $(TARGETS)

bin/%: %.c
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
	@echo "Built $@"
```

或者使用 `addprefix` 自动加前缀：

```makefile
SRCS = helloworld.c para_pass.c
# 将 .c 替换为无后缀，并加上 bin/ 前缀
TARGETS = $(addprefix bin/, $(basename $(SRCS)))

all: $(TARGETS)

bin/%: %.c
	@mkdir -p bin
	$(CC) $(CFLAGS) -o $@ $< $(LDLIBS)
	@echo "Built $@"
```

这里把 `mkdir -p bin` 直接放到了规则里（每编译一个文件执行一次），这样可以保证即使你删除了 `bin` 目录，再次 `make` 时也能自动重建，不会因为找不到目录而报错。

##### ⚠️ 清理命令也要跟着改

别忘了把你的 `clean` 规则也改一下：

```makefile
clean:
	rm -rf bin
```
或者：
```makefile
clean:
	rm -f $(TARGETS)
```

这样你的源码在根目录，编译出的可执行文件全部整洁地躺在 `./bin` 里了。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q194"></a>

## #194 · 刚才的Makefile文件改一下，编译结果都放在bin/目录中

> 🕒 2026-10-26 21:33 ｜ DeepSeek 消息 ID 387–388 ｜ 主题：Makefile 与项目目录结构

**提问**

刚才的Makefile文件改一下，编译结果都放在bin/目录中

**回答**

为了把所有编译产物（包括 `.o` 中间文件和最终的可执行文件）都放到 `bin/` 目录中，我们需要对 Makefile 做两处调整：

1. 加上 `-B` 选项或者利用 `mkdir -p` 确保 `bin/` 目录存在。
2. 修改目标文件和可执行文件的生成路径。

这里提供两个版本的 Makefile，你可以根据偏好选择：

##### 版本 1：标准且清晰（只把可执行文件放 bin 目录，.o 文件放根目录）
如果你只希望最终的可执行文件在 `bin/` 里，中间文件 `.o` 还在当前目录，可以这样写：

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0

TARGET = bin/test_seqlist
OBJS = main.o seqlist.o seqlist_init.o

all: $(TARGET)

$(TARGET): $(OBJS)
	@mkdir -p bin
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c seqlist.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf bin $(OBJS)

.PHONY: all clean
```

---

##### 版本 2：彻底分离（.o 文件和可执行文件全部放进 bin 目录）
如果你希望目录非常整洁，连 `.o` 文件都不想放在根目录，可以这样写：

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0

# 目标可执行文件放在 bin 下
TARGET = bin/test_seqlist

# 源文件列表
SRCS = main.c seqlist.c seqlist_init.c

# 将源文件后缀替换为 .o，并加上 bin/ 前缀
OBJS = $(patsubst %.c, bin/%.o, $(SRCS))

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

# 编译规则：在 bin 目录下生成 .o 文件
bin/%.o: %.c seqlist.h
	@mkdir -p bin
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf bin

.PHONY: all clean
```

##### 💡 关键修改点说明：
1. **`bin/%.o: %.c`**：利用模式规则匹配，让 `main.c` 编译成 `bin/main.o`。
2. **`@mkdir -p bin`**：在命令行前加上 `@` 符号可以隐藏该条命令的回显，让终端输出更干净。`-p` 参数确保即使目录已存在也不会报错。
3. **`rm -rf bin`**：清理时直接删掉整个 `bin` 目录，简单粗暴且有效。

建议你直接使用 **版本 2**，这样你的项目根目录下只会有 `.c`、`.h` 和 `Makefile`，所有的编译产物都整齐地收纳在 `bin/` 中。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q195"></a>

## #195 · 可执行文件在bin/目录，中间文件.o在obj/目录

> 🕒 2026-10-26 21:34 ｜ DeepSeek 消息 ID 389–390 ｜ 主题：Makefile 与项目目录结构

**提问**

可执行文件在bin/目录，中间文件.o在obj/目录

**回答**

这是一个非常标准且专业的 C 语言工程目录结构。我们需要将源文件编译成 `.o` 放到 `obj/` 目录，最后链接成可执行文件放到 `bin/` 目录。

同时，为了防止因为修改头文件（`.h`）导致 Make 没有重新编译依赖它的 `.c` 文件，我加上了自动生成依赖关系的机制（`-MMD -MP`），这是工业界的标准做法。

##### 完整的 `Makefile`

```makefile
CC = gcc
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0
# -MMD -MP 用于自动生成头文件依赖关系
CFLAGS += -MMD -MP

# 定义目录
BIN_DIR = bin
OBJ_DIR = obj

# 最终目标
TARGET = $(BIN_DIR)/test_seqlist

# 源文件和目标文件（利用 patsubst 自动生成路径）
SRCS = main.c seqlist.c seqlist_init.c
OBJS = $(patsubst %.c, $(OBJ_DIR)/%.o, $(SRCS))

# 默认目标
all: $(TARGET)

# 链接规则：将所有的 .o 链接成可执行文件
$(TARGET): $(OBJS)
	@mkdir -p $(BIN_DIR)
	$(CC) $(CFLAGS) -o $@ $^

# 编译规则：把 .c 编译成 obj/ 下的 .o
$(OBJ_DIR)/%.o: %.c
	@mkdir -p $(OBJ_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

# 清理规则
clean:
	rm -rf $(BIN_DIR) $(OBJ_DIR)

# 包含自动生成的依赖文件（.d 文件）
-include $(OBJS:.o=.d)

.PHONY: all clean
```

---

##### 💡 Makefile 亮点解析

1. **`OBJS = $(patsubst %.c, $(OBJ_DIR)/%.o, $(SRCS))`**
   这行代码非常关键。它会自动把 `main.c seqlist.c seqlist_init.c` 转换成 `obj/main.o obj/seqlist.o obj/seqlist_init.o`。
2. **`@mkdir -p $(OBJ_DIR)` 和 `@mkdir -p $(BIN_DIR)`**
   在编译和链接前，自动创建对应的文件夹。`@` 符号会隐藏命令本身的输出，让终端看起来更干净。`-p` 确保目录存在时不会报错。
3. **`-MMD -MP`（黑科技：自动依赖追踪）**
   如果你修改了 `seqlist.h`，`main.c` 和 `seqlist.c` 都应该重新编译。加上这两个参数后，GCC 会自动生成 `obj/*.d` 文件，里面记录了“谁包含了谁”。Makefile 底部的 `-include $(OBJS:.o=.d)` 会将它们引入。这样你修改头文件后，`make` 就能自动识别并重新编译受影响的文件。

---

##### 📁 编译后的目录结构

执行 `make` 后，你的项目目录会变成这样：

```text
你的项目目录/
├── bin/
│   └── test_seqlist      <-- 最终可执行文件
├── obj/
│   ├── main.o
│   ├── main.d            <-- 自动生成的依赖文件
│   ├── seqlist.o
│   ├── seqlist.d
│   ├── seqlist_init.o
│   └── seqlist_init.d
├── main.c
├── seqlist.c
├── seqlist_init.c
├── seqlist.h
└── Makefile
```

如果你想清理所有编译产物，只需执行 `make clean`，它会干净利落地把 `bin/` 和 `obj/` 删掉。

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---

<a id="q196"></a>

## #196 · 常用的是将.c文件放到src/目录，*.h文件放到include目录吗

> 🕒 2026-10-26 21:37 ｜ DeepSeek 消息 ID 391–392 ｜ 主题：Makefile 与项目目录结构

**提问**

```text
常用的是将.c文件放到src/目录，*.h文件放到include目录吗
```

**回答**

**是的，这绝对是 C 语言工程中最经典、最常用的标准目录结构之一。**

你提到的这种结构通常被称为 **“分离头文件与源文件”** 结构，在 Linux 开源社区（如 Git、Redis 等）、嵌入式开发以及大型 C 项目中极为普遍。

一个标准的项目目录通常长这样：

```text
project_root/
├── include/       <-- 存放所有公开的 .h 头文件
│   └── seqlist.h
├── src/           <-- 存放所有 .c 源文件
│   ├── main.c
│   ├── seqlist.c
│   └── seqlist_init.c
├── obj/           <-- 编译生成的中间文件 (.o)
├── bin/           <-- 编译生成的可执行文件
└── Makefile
```

##### 💡 为什么要这样分？

1. **接口与实现分离**：`include/` 暴露的是“程序能做什么”（API 声明），`src/` 隐藏的是“程序怎么做”（具体实现）。别人用你的库，只需要看 `include/`。
2. **避免头文件冲突**：把所有头文件集中在一个目录，方便在 Makefile 中用 `-Iinclude` 统一指定头文件搜索路径。
3. **源码干净**：根目录下没有乱七八糟的 `.c` 和 `.h` 混杂，一目了然。

---

##### 如果把结构改成这样，Makefile 怎么改？

既然目录变了，在 `#include "seqlist.h"` 时，编译器可能找不到头文件（因为头文件不在当前 `.c` 文件同目录下）。我们需要在 Makefile 里加上 `-Iinclude` 参数告诉编译器去哪里找头文件。

根据你的新目录结构，更新后的 **Makefile** 如下：

```makefile
CC = gcc
# 关键点：添加 -Iinclude 指定头文件搜索路径
CFLAGS = -std=c99 -pedantic-errors -Wall -Wextra -Werror -g -O0 -Iinclude
CFLAGS += -MMD -MP

BIN_DIR = bin
OBJ_DIR = obj
TARGET = $(BIN_DIR)/test_seqlist

# 自动找出 src/ 下所有的 .c 文件
SRCS = $(wildcard src/*.c)
# 将 src/xxx.c 替换为 obj/xxx.o
OBJS = $(patsubst src/%.c, $(OBJ_DIR)/%.o, $(SRCS))

all: $(TARGET)

$(TARGET): $(OBJS)
	@mkdir -p $(BIN_DIR)
	$(CC) $(CFLAGS) -o $@ $^

# 编译规则：把 src/ 下的 .c 编译成 obj/ 下的 .o
# $< 代表 src/xxx.c，$@ 代表 obj/xxx.o
$(OBJ_DIR)/%.o: src/%.c
	@mkdir -p $(OBJ_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf $(BIN_DIR) $(OBJ_DIR)

-include $(OBJS:.o=.d)

.PHONY: all clean
```

##### 📝 源码需要改的地方

既然头文件挪到了 `include/` 目录，在你的 `src/*.c` 文件里，包含头文件的方式依然可以是：

```c
#include "seqlist.h"
```

因为我们在 Makefile 里加了 `-Iinclude`，编译器在找 `"seqlist.h"` 时，会优先去当前 `src/` 目录找，找不到就会去 `-I` 指定的 `include/` 目录找。

##### 🌟 进阶结构（更大型的项目）

如果你的项目越来越大，人们还会继续细分：
*   把 `main.c` 挪到单独的 `test/` 或 `app/` 目录。
*   把公共库移到 `lib/` 目录。
*   把第三方依赖放到 `third_party/` 或 `vendor/` 目录。

但对于你目前学习的规模，`include/` + `src/` + `bin/` + `obj/` 已经是非常专业、干净的工程结构了。建议你以后写代码都采用这种结构！

[↑ 回到顶部](#top) ｜ [« 返回总索引](./README.md)

---
