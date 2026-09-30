
## 什么是时钟？
简单来说，时钟是具有周期性的脉冲信号，最常用的是占空比为50%的方波。时钟是单片机的脉搏源，单片机一般会通过时钟来控制内部工作。搞懂时钟的走向及关系，对使用单片机非常重要。

## STM32F1的时钟树
|时钟源名称|频率|材料|用途|
|:---:|:---:|:---:|:---:|
|高速外部振荡器(HSE)|4-16MHz|晶体/陶瓷|SYSCLK/RTC|
|低速外部振荡器(LSE)|32.768KHz| 晶体/陶瓷|RTC|
|高速内部振荡器(HSI)|8MHz| RC|SYSCLK|
|低速内部振荡器(LSI)|40KHz| RC|RTC/IWDG|
I：Internal， E：External

### STM32F03时钟树简图

时钟源、锁相环: `HAL_RCC_OscConfig()`
系统时钟、总线：`HAL_RCC_ClockConfig()`
使能外设时钟: `__HAL_RCC_PPP_CLK_ENABLE()` 宏
扩展外设时钟(RTC/ADC/USB): `__HAL_RCCEx_PeriphCLKConfig()` 

## 系统时钟配置步骤
### 配置系统时钟
1. 配置HSE_VALUE
它的作用是高速HAL库外部晶振频率。这个宏在stm32xxxx_hal_conf.h中定义。
2. 调用SystemInit()函数（可选）。在启动文件中调用，在system_stm32xxxx.c中定义
3. 选择时钟源，配置PLL。通过HAL_RCC_OscConfig()函数设置。
4. 选择系统时钟源，配置总线分频器,pin。通过HAL_RCC_ClockConfig()函数设置。
5. 配置扩展外设时钟（可选）。通过HAL_RCCEx_PeriphCLKConfig()函数设置。
其中3+4+5可以封装为一个函数，比如sys_stm32_clock_init()

### 外设时钟的使能和失能
要使用某个外设，必须先使能该外设的时钟。如，__HAL_RCC_GPIOA_CLK_ENABLE() 使能GPIOA的时钟
要关闭某个外设，必须先关闭该外设的时钟。如，__HAL_RCC_GPIOA_CLK_DISABLE() 关闭GPIOA的时钟

### F1系统时钟配置
1. HAL_RCC_OscConfig()函数
2. HAL_RCC_ClockConfig()函数
