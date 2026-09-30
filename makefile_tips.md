1.make的技巧
打印Makefile的规则和变量：`make -p`
可以把make命令规则和变量存入文件：`make -p>./rules.txt`
使用vi中的替换命令`:g/^#/d`将以#开头的行删掉

2. 编译器Warning清零
开启编译器的最高Warning级别，把所有Warning当Error处理，零Warning才算通过
CFLAGS += -Wall -Wextra -Wreturn-type -Wuninitialized 
CFLAGS += -Wshadow -Wundef -Wformat=2
CFLAGS += -Werror

```text
1. -Wall
启用一组经过筛选的高价值常用警告。虽然名字带有 "All"，但它并不是打开所有警告，而是能捕获绝大多数容易被忽略的逻辑错误、未定义行为和潜在缺陷（如未初始化的变量、返回值类型不匹配、括号缺失导致的优先级误解等）。
2. -Wextra
启用 -Wall 未包含但同样实用的额外警告检查。例如未使用的函数参数、局部变量遮蔽外层同名变量、switch 语句中隐式堕落（忘记写 break）等。
3. -Wreturn-type
当函数声明了返回值类型，但实际代码中存在不带返回值的 return 语句，或者函数定义了返回类型但默认类型是 int 时，编译器会发出警告。
4. -Wuninitialized
当自动变量在初始化之前就被使用时，编译器会发出警告。这有助于避免读取到内存中的随机垃圾值。
5. -Wshadow
当局部变量声明了一个与外层作用域同名变量相同的名称（即变量遮蔽）时，编译器会发出警告，防止因同名引发不必要的逻辑麻烦。
6. -Wundef
当 #if 预处理指令中使用了未定义的宏时，编译器会发出警告（默认情况下未定义的宏会被当作 0 处理，这可能掩盖潜在的逻辑错误）。
7. -Wformat=2
启用比 -Wall 中更高级别的格式化字符串检查。它不仅检查 printf 和 scanf 等函数的参数类型与格式串是否匹配，还会检查非字面量的格式串、格式串安全问题（如 format-security）以及 Y2K 问题等，防范格式化字符串漏洞。
8. -Werror
将所有已启用的编译警告升级为错误（fatal error）。一旦代码触发了上述任何一个警告，编译过程将直接失败并停止。这通常用于 CI/CD 流水线或发布构建中，强制要求代码零警告，防止潜在隐患被忽略。
总结：
这套编译选项组合构成了非常严格的代码质量防线。它通过 -Wall 和 -Wextra 覆盖了大部分常见的代码缺陷，通过 -Wshadow、-Wundef 和 -Wformat=2 进行了更细致的静态检查，最后通过 -Werror 强制开发者在编译阶段修复所有发现的问题。

```

3. cppcheck 静态代码分析工具
cppcheck是免费的嵌入式友好的静态分析工具，能发现运行时才能暴露的问题
```bash
cppcheck --enable=all --inconclusive --std=c11 src/ 2>&1 |grep -v "^$"
```

4. 单元测试：将逻辑从硬件中剥离出来
5. 