# i2c 驱动笔记 #

## i2c 总线简介 ##

i2c(IIC, 读作/ˈaɪ skwɛərd ˈsiː/或者/ˈaɪ tuː ˈsiː/) 的意思是inter IC，也就是芯片间通信。i2c 总线是由Philips 在80年代早期设计的，用来实现同一个电路板上的各个芯片之间简易通信的总线。

### i2c 的主要特性 ###

* 只有两条连接线，SDL(串行数据线)和SCL(串行时钟线)
* 没有严格的波特率要求
* 芯片间是简单的主/从关系，每个总线上的设备可以通过一个唯一的地址识别
* i2c 是一个真正的多主总线，提供仲裁逻辑和冲突检测

## i2c 子系统架构 ##

![i2c 子系统架构](https://i2c.wiki.kernel.org/images-i2c/d/df/I2c-layers.png)

* `i2c-client` i2c 总线上的设备驱动
* `i2c-core` 子系统实现的i2c 协议以及其他公共工具函数
* `i2c-algorithm` 通用算法，给i2c-adapter 驱动提供和子系统交互的接口
* `i2c-adapter` 操作特定的i2c 总线硬件，实现核心子系统中提供的总线操作
* `i2c-dev` 用于实现用户空间的i2c 驱动

## i2c 总线驱动 ##

* `struct i2c_adapter;` i2c 总线驱动硬件适配结构体，实现具体总线的操作
* `struct i2c_algorithm;` i2c 总线驱动的具体操作接口，实现新总线操作时需要实现这个具体的硬件操作接口
* `int i2c_add_adapter(struct i2c_adapter *);` 添加总线驱动
* `int i2c_del_adapter(struct i2c_adapter *);` 删除总线驱动

## i2c 设备驱动 ##

* `struct i2c_client;` i2c 设备结构体，包含`struct device`，用于向驱动模型注册设备
* `struct i2c_driver;` i2c 驱动结构体，包含`struct device_driver`，向系统注册驱动
* `int i2c_register_driver(struct module *, struct i2c_driver *);` 注册设备驱动
* `void i2c_del_driver(struct i2c_driver *);` 删除设备驱动

## i2c 相关的资源和链接 ##

* [i2c wiki](http://en.wikipedia.org/wiki/I%C2%B2C)
* [i2c 协议](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=blob;f=Documentation/i2c/i2c-protocol)
* [i2c 驱动架构分析](http://blog.csdn.net/hongjiujing/archive/2009/04/21/4098547.aspx)
