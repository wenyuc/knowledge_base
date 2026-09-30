
## Linux驱动开发的分类
Linux驱动开发可以根据不同的标准进行分类，以下是一些常见的分类分方式：
https://blog.csdn.net/qq_44705488/article/details/123129354

### 1. 按硬件类型分类

1. 字符设备驱动
   控制那些只能一个字节一个字节读写的设备，如I2C设备，SPI设备，LED等。
   数据访问通常按照先后顺序进行，类似于文件操作中的字节流。
   字符设备是像字节流（类似文件）一样被访问的设备。
   对字符设备发出读写请求时，实际的硬件IO操作通常紧接着发生。
   字符设备驱动程序通常至少要实现open, read, write, close等系统条用。
   开发举例：LED驱动

2. 块设备驱动
   控制那些可以从设备的任意位置读写固定长度数据的设备，如硬盘、U盘、SD卡等。
   数据访问具有随机性，通常通过内存缓冲区（cache buffer）进行。
   块设备驱动通常通过文件系统访问，而不是直接通过设备文件。
   开发举例：硬盘驱动是典型的块设备驱动。硬盘驱动需要实现请求队列管理、缓冲区管理、读写操作等功能。具体来说，硬盘驱动需要维护一个请求队列，将来自文件系统的读写请求排队处理。同时，硬盘驱动还需要与硬件寄存器交互，以实现对硬盘硬件的直接控制。在读写操作中，硬盘驱动需要将数据从内存缓冲区传输到硬盘或从硬盘传输到内存缓冲区。

3. 网络设备驱动
   控制那些实现网络通信的设备，如网卡、Wifi、Bluetooth等设备
   不直接对文件进行操作，而是通过专门的网络接口实现数据发送和接收。
   网络设备驱动通常使用一套与数据包传输相关的函数（如socket函数）与内核通信。
   开发举例：网卡驱动是网络设备驱动的典型代表。网卡驱动需要实现网络协议栈的底层接口，包括数据包的发送和接受、错误处理等功能。在发送数据时，网卡驱动将数据包从内核网络协议栈传输到网卡硬件；在接收数据时，网卡驱动讲数据包从网卡硬件传输到内核网络协议栈。此外，网卡驱动嗨需要与硬件寄存器交互，以实现对网卡硬件的直接控制。

4. 其他类型设备驱动 
   USB设备驱动、显示设备驱动、Sound设备驱动、输入设备驱动等。

## 用C语言开发Linux驱动的基本步骤

### 1. 定义设备号和设备结构体
- 字符设备在Linux中通过设备号来唯一标识。设备号由住设备号和次设备号组成。
- 定义一个`cdev`结构体来表示字符设备，该结构体包含设备操作函数集合等。

### 2. 编写设备操作函数
- 实现`file_operations`结构体中的成员函数，如`open, release, read, write`等，这些函数讲直接处理来自用户空间的设备操作请求。
第 1589 行，owner 拥有该结构体的模块的指针，一般设置为 THIS_MODULE。
第 1590 行，llseek 函数用于修改文件当前的读写位置。
第 1591 行，read 函数用于读取设备文件。
第 1592 行，write 函数用于向设备文件写入(发送)数据。
第 1596 行，poll 是个轮询函数，用于查询设备是否可以进行非阻塞的读写。
第 1597 行，unlocked_ioctl 函数提供对于设备的控制功能，与应用程序中的 ioctl 函数对应。
第 1598 行，compat_ioctl 函数与 unlocked_ioctl 函数功能一样，区别在于在 64 位系统上，32 位的应用程序调用将会使用此函数。在 32 位的系统上运行 32 位的应用程序调用的是`unlocked_ioctl`。
第 1599 行，mmap 函数用于将设备的内存映射到进程空间中(也就是用户空间)，一般帧缓冲设备会使用此函数，比如 LCD 驱动的显存，将帧缓冲(LCD 显存)映射到用户空间中以后应用程序就可以直接操作显存了，这样就不用在用户空间和内核空间之间来回复制。
第 1601 行，open 函数用于打开设备文件。
第 1603 行，release 函数用于释放(关闭)设备文件，与应用程序中的 close 函数对应。
第 1604 行，fasync 函数用于刷新待处理的数据，用于将缓冲区中的数据刷新到磁盘中。
第 1605 行，aio_fsync 函数与 fasync 函数的功能类似，只是 aio_fsync 是异步刷新待处理的数据。

字符驱动开发中常用的就是：open、release、write、read函数。
ioctl（Input/Output Control）主要用于执行无法归类为普通读写的设备控制操作。换句话说，read/write 负责传数据，ioctl 负责发命令。
When a 64-bit application calls ioctl(), the kernel executes unlocked_ioctl. When a 32-bit application runs on a 64-bit kernel, the kernel enters compatibility mode and executes compat_ioctl.
compat_ioctl is not a completely independent function; it usually forward calls to unlocked_ioctl after handling 32-bit/64-bit differences. The kernel provides a standard helper function compat_ptr_ioctl() for this purpose :
```c
// Most architectures: directly convert the 32-bit pointer and forward to unlocked_ioctl
long compat_ptr_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    return file->f_op->unlocked_ioctl(file, cmd, (unsigned long)compat_ptr(arg));
}
```


### 3. 注册字符设备
- 使用`alloc_chrdev_region`或`register_chrdev_region`函数为字符设备分配设备号。
- 初始化`cdev`结构体，并将其添加到内核中。

### 4. 创建设备节点
- 在`/dev`目录下创建设备文件，以便用户空间程序可以通过标准文件操作接口(如open,read,write等)来访问设备。
- 可以手动在`/dev`目录下创建设备文件，或者使用`udev`规则来自动创建设备节点，这在现在Linux系统中更为常见和推荐。

### 5. 编写模块加载和卸载函数
- 使用`model_init, model_exit`宏来注册模块加载和卸载函数。

### 6. 编译和加载驱动模块
- 编译驱动程序生成`.ko`文件。
- 使用`insmod`或`modprobe`命令加载驱动模块，`modprobe`命令会自动处理模块依赖关系，因此更为推荐使用。`rmmod`卸载驱动模块。
- module_init() 函数用来向 Linux 内核注册一个模块加载函数，参数 xxx_init 就是需要注册的具体函数，当使用“insmod”命令加载驱动的时候，xxx_init 这个函数就会被调用。

module_exit() 函数用来向 Linux 内核注册一个模块卸载函数，参数 xxx_exit 就是需要注册的具体函数，当使用“rmmod”命令卸载具体驱动的时候 xxx_exit 函数就会被调用。

- modprobe 命令主要智能在提供了模块的依赖性分析、错误检查、错误报告等功能，推荐使用 modprobe 命令来加载驱动。modprobe 命令默认会去/lib/modules/目录中查找模块，如使用Linux kernel 4.4.15版本，modprobe就会再`/lib/modules/4.1.15`目录中查找相应的驱动模块，一般自己制作的根文件系统中是不会有这个目录的，所以需要自己手动创建。
使用 `modprobe` 命令可以卸载掉驱动模块所依赖的其他模块，前提是这些依赖模块已经没有被其他模块所使用，否则就不能使用 `modprobe` 来卸载驱动模块。`modprobe -r drv.ko` 所以对于模块的卸载，推荐使用 rmmod 命令。


### 7. 例子
```c
#include <linux/module.h>   //包含模块头文件
#include <linux/fs.h>       //包含内核头文件
#include <linux/cdev.h>     //包含文件系统头文件
#include <linux/miscdevice.h> //杂项设备
#include <linux/uaccess.h>  //包含用户空间访问头文件
  
#define MY_DEVICE_NAME "my_char_dev"  
  
static int major;  
static struct cdev *my_cdev;  
static struct file_operations my_fops = {  
    .owner = THIS_MODULE,  
    .open = my_open,  
    .release = my_release,  
    .read = my_read,  
    .write = my_write,  
    // 可以添加其他操作函数  
};  
  
static int my_open(struct inode *inode, struct file *file)  
{  
    // 设备打开逻辑  
    return 0;  
}  
  
static int my_release(struct inode *inode, struct file *file)  
{  
    // 设备关闭逻辑  
    return 0;  
}  
  
static ssize_t my_read(struct file *file, char __user *buf, size_t count, loff_t *ppos)  
{  
    // 设备读取逻辑  
    return 0;  
}  
  
static ssize_t my_write(struct file *file, const char __user *buf, size_t count, loff_t *ppos)  
{  
    // 设备写入逻辑  
    return count;  
}  
  
static int __init my_char_dev_init(void)  
{  
    int ret;  
  
    ret = alloc_chrdev_region(&major, 0, 1, MY_DEVICE_NAME);  
    if (ret < 0) {  
        printk(KERN_WARNING "Failed to allocate major number\n");  
        return ret;  
    }  
  
    my_cdev = cdev_alloc();  
    my_cdev->ops = &my_fops;  
    my_cdev->owner = THIS_MODULE;  
  
    ret = cdev_add(my_cdev, MKDEV(major, 0), 1);  
    if (ret < 0) {  
        printk(KERN_WARNING "Failed to add cdev\n");  
        goto error_unregister;  
    }  
  
    // 可以添加创建设备节点的代码  
  
    return 0;  
  
error_unregister:  
    unregister_chrdev_region(MKDEV(major, 0), 1);  
    return ret;  
}  
  
static void __exit my_char_dev_exit(void)  
{  
    cdev_del(my_cdev);  
    unregister_chrdev_region(MKDEV(major, 0), 1);  
    // 可以添加删除设备节点的代码  
}  
  
module_init(my_char_dev_init);  
module_exit(my_char_dev_exit);  
  
MODULE_LICENSE("GPL");  
MODULE_AUTHOR("Your Name");  
MODULE_DESCRIPTION("A simple character device driver");
```

## 3. 如何把一个字符驱动的程序注册到内核中
对于字符设备驱动，驱动模块加载成功后需要注册字符设备，卸载后也需要注销掉字符设备。

字符设备的注册和注销函数原型：
```c
static inline int register_chrdev(unsigned int major, const char *name,
								const struct file_operations *fops)
    作用：用于注册字符设备，有三个参数；
    major：主设备号，Linux 下每个设备都有一个设备号，设备号分为主设备号和次设备号两部分；
    name：设备名字，指向一串字符串；
    fops：结构体 file_operations 类型指针，指向设备的操作函数集合变量。
    返回值：成功返回0，失败返回负数。
static inline void unregister_chrdev(unsigned int major, const char *name)
    作用：用户注销字符设备，此函数有两个参数；
    major：要注销的设备对应的主设备号；
    name：要注销的设备对应的设备名。
    返回值：无。
```
一般字符设备的注册在驱动模块的入口函数 xxx_init 中进行，字符设备的注销在驱动模块的出口函数 xxx_exit 中进行。

### 1. 设备号的组成
Linux提供了一个名为`dev_t`的数据类型表示设备号，`dev_t`定义在文件`include/linux/types.h`中定义。
```c
typedef __u32 __kernel_dev_t;
typedef __kernel_dev_t dev_t;
```
`__u32`定义在文件`include/uapi/asm-generic/int-II64.h`中定义。
```c
typedef unsigned int __u32;
```
这 32 位的数据构成了主设备号和次设备号两部分，其中高 12 位为主设备号，低 20 位为次设备号。因此 Linux系统中主设备号范围为 0~4095，所以在选择主设备号的时候一定不要超过这个范围。

在文件在文件 `include/linux/kdev_t.h` 中提供了几个关于设备号的操作函数(本质是宏 )，如下所示:
```c
 #define MINORBITS 20
 #define MINORMASK ((1U << MINORBITS) - 1) 8 
 #define MAJOR(dev) ((unsigned int) ((dev) >> MINORBITS))
 #define MINOR(dev) ((unsigned int) ((dev) & MINORMASK))
 #define MKDEV(ma,mi) (((ma) << MINORBITS) | (mi))
```


### 2. 定义字符设备结构体
在Linux内核中，通常使用struct cdev来表示字符设备。此外，对于杂项设备，还可以使用struct miscdevice。首先，根据设备需求定义一个包含struct cdev（或struct miscdevice）的结构体。例如：
```c
#include <linux/cdev.h>  
  
struct my_char_dev {  
    struct cdev cdev;  
    // 其他设备特定字段  
};
```

对于杂项设备，定义如下：
```c
#include <linux/miscdevice.h>  
  
struct my_misc_dev {  
    struct miscdevice misc;  
    // 其他设备特定字段  
};
```

## 3. Linux驱动开发中的C语言基础

给位6赋值1
```c
uint32_t value = 0;
value &= 0xFFFFFFBF;
value |= 0x40;

// 或者
// value &= ~(1 << 6);
// value |= (1 << 6);
```
按位异或用于控制位6翻转
```c
uint32_t value = 0;
value ^= (1 << 6);
```
获取位6的值
```c
uint32_t value = 0;
uint32_t bit6 = (value >> 6) & 1;
```
获取位6的值，并设置位6的值为0
```c
uint32_t value = 0;
uint32_t bit6 = (value >> 6) & 1;
value &= ~(1 << 6);
```
获取位6的值，并设置位6的值为1
```c
uint32_t value = 0;
uint32_t bit6 = (value >> 6) & 1;
value |= (1 << 6);
```
