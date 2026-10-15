下面把你用到的 LLDB 命令，一一对应到 GDB 命令。

---

## 一、启动与断点

| LLDB | GDB | 说明 |
|------|-----|------|
| `lldb test_stack` | `gdb test_stack` | 加载可执行文件 |
| `b main` | `b main` | 在 `main` 下断点 |
| `b *0x40010808` | `b *0x40010808` | 在地址下断点 |
| `run hello world` | `run hello world` | 带参数运行 |
| `continue` / `c` | `continue` / `c` | 继续运行 |
| `quit` / `q` | `quit` / `q` | 退出调试器 |

---

## 二、单步执行

| LLDB | GDB | 说明 |
|------|-----|------|
| `si` | `si` | 汇编级单步步入 |
| `ni` | `ni` | 汇编级单步步过 |
| `s` | `s` / `step` | 源码级步入 |
| `n` | `n` / `next` | 源码级步过 |
| `finish` | `finish` | 执行到当前函数返回 |
| `until` | `until` | 执行到指定行 |

---

## 三、寄存器

| LLDB | GDB | 说明 |
|------|-----|------|
| `register read` | `info registers` | 读所有通用寄存器 |
| `register read rdx` | `info registers rdx` | 读指定寄存器 |
| `re r rdx` | `p $rdx` | 读 `rdx` 的简写 |
| `register read edx` | `info registers edx` | 读低 32 位 |
| `register write rip 0x...` | `set $rip = 0x...` | 修改寄存器 |
| `p/x $rdx` | `p/x $rdx` | 用表达式读寄存器 |

---

## 四、内存

| LLDB | GDB | 说明 |
|------|-----|------|
| `me read -fx -s8 -c1 0x...` | `x/1gx 0x...` | 读 8 字节十六进制 |
| `me read -fd -s4 -c1 0x...` | `x/1dw 0x...` | 读 4 字节十进制 |
| `me read -fs 0x...` | `x/s 0x...` | 读字符串 |
| `me read -fx -s8 -c4 0x...` | `x/4gx 0x...` | 读 4 个 8 字节 |
| `memory write 0x... 0x41` | `set {char}0x... = 0x41` | 写内存 |

**GDB 的 `x` 命令格式**：
```
x/[数量][格式][单位] 地址
```
| 格式 | 含义 | 单位 | 含义 |
|------|------|------|------|
| `x` | 十六进制 | `b` | 1 字节 |
| `d` | 十进制 | `h` | 2 字节 |
| `s` | 字符串 | `w` | 4 字节 |
| `i` | 指令 | `g` | 8 字节 |

**示例**：
- `x/1gx 0x...` = 读 1 个 8 字节，十六进制
- `x/4gx 0x...` = 读 4 个 8 字节，十六进制
- `x/s 0x...` = 读字符串

---

## 五、反汇编

| LLDB | GDB | 说明 |
|------|-----|------|
| `di` | `disassemble` | 反汇编当前函数 |
| `di -p` | `disassemble /r $pc` | 反汇编当前帧 |
| `di -s 0x...` | `disassemble 0x...` | 从指定地址反汇编 |
| `di -n main` | `disassemble main` | 反汇编指定函数 |
| `di -c 20` | `disassemble /r` + `x/20i $pc` | 反汇编 20 条指令 |

---

## 六、栈帧与调用栈

| LLDB | GDB | 说明 |
|------|-----|------|
| `bt` | `bt` / `backtrace` | 显示调用栈 |
| `frame select 0` | `frame 0` | 切换到指定帧 |
| `up` | `up` | 上一层帧 |
| `down` | `down` | 下一层帧 |
| `frame variable` | `info locals` | 显示当前帧局部变量 |
| `frame variable argc argv` | `p argc` / `p argv` | 显示指定变量 |
| `p argc` | `p argc` | 打印变量 |
| `p argv[0]` | `p argv[0]` | 打印数组元素 |

---

## 七、表达式与类型

| LLDB | GDB | 说明 |
|------|-----|------|
| `p/x $rdx` | `p/x $rdx` | 十六进制打印 |
| `p sizeof(int)` | `p sizeof(int)` | 打印类型大小 |
| `ptype argv` | `ptype argv` | 打印类型 |
| `expression $rdx` | `p $rdx` | 求值表达式 |

---

## 八、stop-hook（自动命令）

| LLDB | GDB | 说明 |
|------|-----|------|
| `target stop-hook add` | `commands` | 给断点添加自动命令 |
| `target stop-hook list` | `info breakpoints` | 查看断点命令 |
| `target stop-hook delete` | `delete` | 删除断点命令 |

**GDB 的等价写法**：
```
(gdb) break main
(gdb) commands
> info registers rdx rsi rdi
> continue
> end
```

---

## 九、你之前用到的命令对照

| 你的 LLDB 命令 | GDB 等价命令 |
|---------------|-------------|
| `lldb test_stack` | `gdb test_stack` |
| `b main` | `b main` |
| `run hello world` | `run hello world` |
| `si` | `si` |
| `re r rdx` | `p $rdx` |
| `me read -fx -s8 -c1 0x...` | `x/1gx 0x...` |
| `me read -fs 0x...` | `x/s 0x...` |
| `me read -fx -s8 -c4 0x...` | `x/4gx 0x...` |
| `di -p` | `disassemble /r $pc` |
| `bt` | `bt` |
| `frame variable argc argv` | `p argc` + `p argv` |
| `p argv[0]` | `p argv[0]` |
| `quit` | `quit` |

---

## 十、总结

| 功能 | LLDB | GDB |
|------|------|-----|
| 读寄存器 | `register read rdx` | `info registers rdx` |
| 读内存 | `me read -fx -s8 -c1 0x...` | `x/1gx 0x...` |
| 读字符串 | `me read -fs 0x...` | `x/s 0x...` |
| 反汇编 | `di -p` | `disassemble /r $pc` |
| 调用栈 | `bt` | `bt` |
| 局部变量 | `frame variable` | `info locals` |
| 打印变量 | `p argc` | `p argc` |
| 单步 | `si` | `si` |

**核心区别**：
- **LLDB**：命令更"啰嗦"，但语义清晰（`register read`、`memory read`）
- **GDB**：命令更"短"，但有记忆负担（`x/1gx`、`info registers`）
- **两者概念一一对应**，学会一个就能快速迁移到另一个

**最常用的对照**：
```lldb
# LLDB
(lldb) register read rdx
(lldb) me read -fx -s8 -c1 0x7fffffffe338
(lldb) di -p
```

```gdb
# GDB
(gdb) info registers rdx
(gdb) x/1gx 0x7fffffffe338
(gdb) disassemble /r $pc
```