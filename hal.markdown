# Android HAL #

Android Hal 是用来隔离linux内核和Android用户空间的一个中间库，它主要是为了让一些硬件厂商不用根据linux内核的gpl协议放出全部的linux驱动代码。有了hal硬件厂商就可以只在内核驱动中实现基本的操作而把硬件操作的逻辑移到hal层来实现，hal层是apache协议的，所以厂商可以只提供二进制库。

## HAL Arch ##

hal的调用流程如下：

app -> framework -> jni -> runtime lib -> libhardware -> hal stub -> hal imp -> kernel driver

## HAL implementations ##

Android中HAL的实现有两种方式：

- framework -> jni -> device driver，直接通过jni操作硬件驱动
- framework -> jni -> libhardware -> hal stub -> device driver，通过hal stub操作硬件驱动

第一种方式通过jni直接操作硬件驱动，但这样做可移植性很差，因为没有一个统一的中间api，framework调用的接口和硬件驱动接口都是可变的。

第二种方式则通过hal stub来给上层提供一个统一的调用接口，只要在hal中实现这些接口，用户层的代码就可以不用做太大的改变。

## HAL api ##

新的stub hal的实现和它提供的api(hardware/libhardware)。它的include文件夹如下：

### 头文件 ###

`include/hardware`

### 结构体 ###

- `struct hw_module_t;`
- `struct hw_module_methods_t;`
- `struct hw_device_t;`

### 主要接口 ###

`int hw_get_module(const char *id, const struct hw_module_t **module);`

这个接口的主要工作是通过id和系统的设置获取到硬件共享库的路径，然后通过dlopen打开共享库并通过一个固定的符号找到`struct hw_module_t`在内存中的地址，然后再通过该结构体提供的方法来打开设备并操作设备。

## HAL in power manager ##
## HAL example ##
