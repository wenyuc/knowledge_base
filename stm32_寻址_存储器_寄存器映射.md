
## STM32的寻址范围
- 32位的单片机可以有32根地址线(每根地址线有两种状态：导通或不导通)。
- 单片机内存地址访问的存储单元是按字节编址的，即一个字节占一个地址，而不是按bit编址的。
- STM32寻址大小：$2^{32}=4G(字节，B)$
- STM32寻址范围：0x0000 0000 ~ 0xFFFF FFFF

## 存储器映射
存储器指可以存储数据的设备，本身没有地址信息，对存储器分配地址的过程称为存储器映射。

如，一款星忆的存储器，有19根地址线：A0 - A18， 16根数据线：D0 - D15. 那么它有$2^{19}=512K$个地址，每个地址占16bit，即2字节，那么存储器大小为$512K*2=1M$字节。

它的地址范围0-512K，即0x0000 0000 ~ 0x0007 FFFF。 或0x0000 0001 ~ 0x0008 0000

## 存储器功能划分（F1为例）
ST将4GB的地址空间分为8块，如下

|存储块|功能|地址范围|
|:---:|:---:|:---:|
|Block 0|Code(FLASH)|0x0000 0000 ~ 0x1FFF FFFF(512M)|
|Block 1|SRAM|0x2000 0000 ~ 0x3FFF FFFF(512M)|
|Block 2|片上外设|0x4000 0000 ~ 0x5FFF FFFF(512M)|
|Block 3|FSMC Bank1&2|0x6000 0000 ~ 0x7FFF FFFF(512M)|
|Block 4|FSMC Bank3&4|0x8000 0000 ~ 0x9FFF FFFF(512M)||
|Block 5|FSMC寄存器|0xA000 0000 ~ 0xBFFF FFFF(512M)|
|Block 6|未用到|0xC000 0000 ~ 0xDFFF FFFF(512M)|
|Block 7|Cortex M3内部外设|0xE000 0000 ~ 0xFFFF FFFF(512M)|

### Block 0 功能划分
|存储块|功能|地址范围|
|:---:|:---:|:---:|
|Block 0|FLASH或系统存储器别名区|0x0000 0000 ~ 0x0007 FFFF(512K)|
||保留|0x0008 0000 ~ 0x07FF FFFF|
||用户FLASH，用于存储用户代码|0x0800 0000 ~ 0x0807 FFFF(512K)|
||保留|0x0808 0000 ~ 0x1FFF EFFF|
||系统存储器，存储出厂Bootloader|0x1FFF F000 ~ 0x1FFF F7FF(2K)||
||选项字节，配置读保护等|0x1FFF F800 ~ 0x1FFF F80F(16B)|
||保留|0x1FFF F810 ~ 0x1FFF FFFF|

### Block1 功能划分
|存储块|功能|地址范围|
|:---:|:---:|:---:|
|Block 1|SRAM|0x2000 0000 ~ 0x2000 FFFF(64K)|
||保留|0x2001 0000 ~ 0x3FFF FFFF|

### Block2 功能划分
|存储块|功能|地址范围|
|:---:|:---:|:---:|
|Block 2|APB1总线外设|0x4000 0000 ~ 0x4000 77FF|
||保留|0x4000 7800 ~ 0x4000 FFFF|
||APB2总线外设|0x4001 0000 ~ 0x4001 3FFF|
||保留|0x4001 4000 ~ 0x4001 7FFF|
||AHB总线外设|0x4001 8000 ~ 0x4002 33FF|
||保留|0x4002 3400 ~ 0x5FFF FFFF|

...

## 寄存器映射
寄存器是单片机内部一种特殊的内存，可以实现对单片机各个功能的控制。简单来说，寄存器就是单片机内部的控制机构。

### STM32寄存器分类
|大类|小类|说明|
|:---:|:---:|:---:|
|内核寄存器|内核相关寄存器|包含R0～R15，xPSR，特殊功能寄存器等|
||中断控制寄存器|包含NVIC和SCB相关寄存器，NVIC有: ISER, ICER,ISPR,IP等；SCB有:VTOR,AIRCR,SCR等|
||SysTick寄存器|包含CTRL、LOAD、VAL和CALIB四个寄存器|
||内存保护寄存器|可选功能，STM32F103没有|
||调试系统寄存器|ETM，ITM，DWT，IPIU等相关寄存器|
|外设寄存器||包含GPIO, UART,IIC,SPI,TIM,DMA,RTC,PWR,CAN,USB等各种外设寄存器|

### 寄存器映射
寄存器映射，即寄存器地址和存储器地址的映射关系。或者说，给寄存器地址命名的过程，就叫寄存器映射。
GPIOA_ODR寄存器地址的计算过程：
- 获取外设挂在哪个总线上面？查原理图，得挂在APB2总线上
- 获取总线基地址，APB2总线基地址为0x4001 0000
- 获取外设地址偏移量，GPIOA相对APB2总线基地址的偏移量为0x800

那么，寄存器地址0x4001080C，映射后（给寄存器地址命名后）为GPIOA_ODR

字节使用寄存器映射举例:
```c
 *(unsigned int *)0x4001080C = 0xFFFF;
```
定义一个名字后再操作：
```c
#define GPIOA_ODR *(unsigned int *)0x4001080C
GPIOA_ODR = 0xFFFF;
```

或者使用结构体，方便的完成对寄存器的映射
```c
/* __IO 不是标准 C 关键字，它是编译器/库定义的宏。在 ARM 嵌入式里，绝大多数情况下它等价于 volatile。
 __IO 在 __IO uint32_t BRR; 中的作用是：把变量声明为“易失性”访问，通常展开后就是 volatile
 如果 BRR 不是 volatile，编译器可能认为 BRR 在循环里没变，直接优化成死循环或只读一次。
 加了 volatile 后，编译器每次都从该地址重新读取。编译器不能随便优化对它的访问；它通常映射到某个固定硬件地址。*/
typedef struct
{
    __IO uint32_t CRL;
    __IO uint32_t CRH;
    __IO uint32_t IDR;
    __IO uint32_t ODR;
    __IO uint32_t BSRR;
    __IO uint32_t BRR;
    __IO uint32_t LCKR;
} GPIO_TypeDef;
#define GPIO_BASE  (0x40010800UL)
#define GPIOA ((GPIO_TypeDef *)GPIO_BASE)
// 或者
// #define GPIOA ((GPIO_TypeDef *)(0x40010800))
//&GPIOA->CRL: 0x40010800
//&GPIOA->CRH: 0x40010804
//&GPIOA->IDR: 0x40010808
//&GPIOA->ODR: 0x4001080C

GPIOA->ODR = 0xFFFF;
```
在`stm32f103xe.h`中定义。