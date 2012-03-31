# USB 驱动程序 #

USB 是为了替代许多不同的低速总线(包括并行，串行和键盘链接)，它有规范的协议，能在多种场合发挥作用，再加上设计上天生的热插拔能力，使得它成为一个便利的低成本的链接总线。

Linux内核主要支持两种类型的USB驱动：Host系统驱动和Gadget驱动。

## USB设备基础 ##

USB 拓朴结构

![USB_Topology](https://github.com/vmlinz/my_notes/raw/master/usb_topology.jpeg)

* 主机控制器 有两种标准，OHCI(Compaq)和UHCI(Intel)
* 端点 端点是从主机到设备或者设备到主机的数据传输通道
 * 控制
 * 中断
 * 批量
 * 等时
* 接口 USB接口只处理一种USB逻辑连接
* 配置 USB接口被绑定为配置
