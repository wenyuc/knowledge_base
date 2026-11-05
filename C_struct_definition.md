
### 结构的定义
如果定义为：
```c
#define MAXSIZE 100
struct Seq {
    ElemType data[MAXSIZE];
    int length;
};
```
那么, 在后续定义变量时，必须每次都写` struct Seq list;`（加上 `struct` 关键字）如：
```c
void initList(struct Seq *L)
{
    L->length=0;
}

int main(void)
{
    //Declare a sequence and initialize
    struct Seq list;
    initList(&list);
    printf("initialize list success, length=%d\n", list.length);
    printf("allicated memory size: %zu bytes.\n", sizeof(list.data));
    return 0;
}
```
如果改用 typedef，代码会更简洁：
```c
typedef struct {
    ElemType data[MAXSIZE];
    int length;
} Seq;

void initList(Seq *L)
{
    L->length=0;
}

int main(void)
{
    //Declare a sequence and initialize
    Seq list;
    initList(&list);
    printf("initialize list success, length=%d\n", list.length);
    printf("allicated memory size: %zu bytes.\n", sizeof(list.data));
    return 0;
}
```
