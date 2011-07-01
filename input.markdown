# Linux Input System #

内核的输入子系统将内核中各种零散的输入驱动整合起来，并给它们提供统一的用户层接口和驱动接口。输入子系统主要由Input Device Driver 和Input Event Driver 以及Input Core三个部分组成。

## Input system summary ##

* Input Device Driver 负责和具体的输入硬件驱动交互，根据硬件信号生成硬件事件
* Input Event Driver 负责将硬件事件转换为用户空间可以识别的事件信号，和用户空间交互
* Input Core 和其他子系统一样，核心模块负责提供公共算法

## Input Device driver ##

Device Driver 和硬件交流，生成硬件事件给Input Core。主要的底层驱动有Serio, USB Hid, Bluetooth Hidp, SPI, ISA/LPC, GPIO等。涵盖了几乎所有的输入硬件。

## Input Event driver ##

## User space ##
