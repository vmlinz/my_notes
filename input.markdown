# Linux Input System #

内核的输入子系统将内核中各种零散的输入驱动整合起来，并给它们提供统一的用户层接口和驱动接口。输入子系统主要由Input Device Driver 和Input Event Driver 以及Input Core三个部分组成。

## Input system summary ##

* Input Device Driver 负责和具体的输入硬件驱动交互，根据硬件信号生成硬件事件
* Input Event Driver 负责将硬件事件转换为用户空间可以识别的事件信号，和用户空间交互
* Input Core 和其他子系统一样，核心模块负责提供公共算法

## Input Device driver ##

Device Driver 和硬件交流，生成硬件事件给Input Core。主要的底层驱动有Serio, USB Hid, Bluetooth Hidp, SPI, ISA/LPC, GPIO等。涵盖了几乎所有的输入硬件。

* `struct input_dev;` 输入设备结构体
* `int __must_check input_register_device(struct input_dev *);` 注册输入设备
* `void input_unregister_device(struct input_dev *);` 注销输入设备
* `void input_event(struct input_dev *dev, unsigned int type, unsigned int code, int value);` 向input core 发送硬件事件

## Input Event driver ##

Event Driver 给用户空间提供了一种与硬件无关的抽象方式来实现和输入硬件的交互。主要的Event Driver 有Evdev 接口，它是一个通用的输入事件驱动。内核中还有其他种类的统一事件驱动。

* `struct input_handler;` 输入事件驱动结构体，实现输入事件的处理函数
* `struct input_handle;` 将输入事件驱动和输入设备驱动结合起来
* `struct input_event;` 输入事件结构体
* `int __must_check input_register_handler(struct input_handler *);` 输入事件驱动注册
* `void input_unregister_handler(struct input_handler *);` 输入事件驱动注销

## Resources and referrences ##

* [Input Programming](http://www.mjmwired.net/kernel/Documentation/input/input-programming.txt)
* [Linux Input Drivers](http://www.mjmwired.net/kernel/Documentation/input/input.txt)
* [Linux 内核Input 子系统分析](http://blog.csdn.net/hongtao_liu/archive/2010/06/18/5679171.aspx)
