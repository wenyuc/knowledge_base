
### Right-Left Rule
在英语编程社区中，这两个概念的命名非常直观，它们的中文翻译正是从英文直译过来的：

*   **数组指针**：**Pointer to an Array** （指向数组的指针）
*   **指针数组**：**Array of Pointers** （存放指针的数组）

---

#### 详细拆解与代码对照

##### 1. 数组指针 (Pointer to an Array)
*   **英语**：`Pointer to an Array`
*   **代码**：`int (*p)[13];`
*   **英文语境描述**：`p is a pointer to an array of 13 integers.`
*   **核心特征**：它本身是一个**指针**（占 8 字节），只是它指向的东西是一整个数组。它的指针步长是**整个数组的大小**（比如 13 × 4 = 52 字节）。

##### 2. 指针数组 (Array of Pointers)
*   **英语**：`Array of Pointers`
*   **代码**：`int *p[13];`
*   **英文语境描述**：`p is an array of 13 pointers to integers.`
*   **核心特征**：它本身是一个**数组**（占 13 × 8 = 104 字节），里面的每一个元素都是指针。它的数组步长是**一个指针的大小**（8 字节）。

---

#### 💡 阅读 C 语言复杂声明的英文口诀

在英语技术文章或 Stack Overflow 中，老程序员经常教初学者一种叫 **"Right-Left Rule"（右左法则）** 的方法来阅读复杂的 C 语言声明。

**规则**：从变量名开始，先向右看，再向左看，遇到括号就掉头。写了pointer就加介词to，写了array就加介词of。

**示例 1：`int (*p)[13];`**
1. 找到变量名 `p`。
2. 向右看，遇到 `)`，掉头向左。
3. 向左看，遇到 `*`。读出：**"p is a pointer to..."** （p 是一个指针，指向...）
4. 出了括号向右看，遇到 `[13]`。读出：**"...an array of 13..."** （包含 13 个元素的数组）
5. 最后向左看，遇到 `int`。读出：**"...integers."**
*   **连起来**：`p is a pointer to an array of 13 integers.`（数组指针）

**示例 2：`int *p[13];`**
1. 找到变量名 `p`。
2. 向右看，遇到 `[13]`。读出：**"p is an array of 13..."** （p 是一个包含 13 个元素的数组）
3. 向左看，遇到 `*`。读出：**"...pointers to..."** （指针，指向...）
4. 继续向左看，遇到 `int`。读出：**"...integers."**
*   **连起来**：`p is an array of 13 pointers to integers.`（指针数组）

**示例3 延伸术语（C 语言面试高频）**
除了这两个，在英语语境下还有两个类似的高频词组，注意不要搞混：
*   **Function Pointer** (函数指针)：`int (*f)();` —— f is a pointer to a function returning int.
*   **Pointer to Function** (与上面同义)：但如果是 **Array of Function Pointers** (函数指针数组)：`int (*f[10])();` —— 包含 10 个函数指针的数组。f is an array of 10 pointers to functions returning int.
*   
函数指针数组的声明，可以写成这样：`int (*f[10])();`，也可以写成这样：`int *f[10];`，它们是等价的。

**示例4 终极BOSS**
`char (*(*x())[])()`
1. 找到变量名 `x`
2. 向右看，遇到 `()`。读出：**"x is a function returning..."**
3. 掉头向左看，遇到`*`。读出：**"...a pointer to..."**, 
4. 继续向右遇到`(`掉头向左，遇到[],读出：**"...an array of ..."**
5. 继续向右遇到`)`，掉头向左，遇到`*`。读出：**"...pointers to..."**
6. 继续向右遇到`(`掉头向左，遇到`()`,读出：**"...functions returning ..."**
7. 掉头遇到char，读出，读出：**"...char"**
把这些连起来：`x is a function returning a pointer to an array of pointers to functions returning char.`
中文翻译：
`x 是一个函数，该函数返回一个指针。这个指针指向一个数组，数组里面的元素是指针，这些指针指向返回 char 的函数。`
其实这个声明在著名的“C 语言声明解析器” cdecl 的官方示例里经常被当作测试用例。

用typedef 来改写：
```c
// 1. 最内层：定义一个函数指针类型
typedef char (*FuncReturningChar)(); 

// 2. 第二层：这是一个指针数组，里面存的都是上面这种函数指针
typedef FuncReturningChar ArrayOfFuncPtrs[];

// 3. 最外层：x 是一个函数，返回一个指针，指向上面那个数组
ArrayOfFuncPtrs *x(); 
```
在 C 语言中，函数不能直接返回一个“函数”（因为函数不是值，不能赋值），函数只能返回指向函数的指针。

所以：
char (*)() 是函数指针。
数组 [] 里装的元素不能是函数本身，只能是函数指针。
x() 返回值也不能是数组本身（C 语言函数不能返回数组），只能返回指向数组的指针。

所以这三层的核心结构其实就是：
函数返回指针 -> 指针指向数组 -> 数组元素是函数指针 -> 这些函数返回 char。

**示例5 `int (*(*f[10])())[13];`**
f is an array of 10 pointers to functions returning a pointer to an array of 13 integers.

**实例6 `char * const *(*next)();`**
next is a pointer to a function returning a pointer to a constant pointer to char. 

```text
next
 └─ *                           pointer to
     └─ ()                      function (no args)
         └─ *                   returning pointer to
             └─ const.          a constant
                 └─ *           pointer to
                     └─ char    char
```

**实例7 `char *(*c[10])(int **p);`**
c is an array of 10 pointers to functions returning a pointer to char, taking a pointer to a pointer to int as argument. 

**实例8 `typedef void(*Handler)(int fd);`**
`typedef`不是定义变量，而是**给一个类型起别名**.
`Handler`是别名，它代表的类型是void (*)(int),通过Right-Left Rule知道它是一个指针,即指向"接受int,返回void的函数"的指针.

几个容易引起混淆的写法：
`void (*Handler)(int);` : 声明+定义,变量Handler为函数指针(分配8字节)，函数返回值为void，参数为int.
如果想让Handler只是"声明"而不定义，可以这样写`extern void (*Handler)(int);` 定义在别处.
`void *Handler(int);` : 声明函数，名为Handler，函数返回值为void *，参数为int.
`void *Handler(int){};` : 声明+定义,定义了一个函数Handler，函数返回值为void，参数为int.且有代码实现，{}
`typedef void Handler(int);` : Handler为别名，代表一个函数类型，函数返回值为void，参数为int
`typedef void *Handler(int);` : Handler为别名，代表一个函数类型，函数返回值为void *,参数为int
