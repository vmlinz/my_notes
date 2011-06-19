# 网络设备驱动基础 #

网络设备驱动和块设备驱动的功能比较类似，都是发送和接收数据包（数据请求）。当然它们实际是有很多不同的，首先块设备在`/dev`目录下有设备节点，而网络设备没有这样的设备入口。read，write等常规的文件接口在网络设备下也没有意义。

最大的区别在于：块设备只响应内核的数据请求；而网络设备驱动要异步地接收来自外部的数据包。简单地说，块设备驱动是被要求传输数据而网络设备是主动请求传输数据。

网络设备驱动还需要支持设置地址，修改传输参数等等这样的操作，所以网络设备驱动的api需要提供这些接口。

## 网络设备注册 ##

* 头文件：`<linux/netdevice.h>`
* `struct net_device` 网络设备结构体
* `struct net_device *alloc_netdev (int size_priv, const char *name,`
				`void (*setup)(struct net_device *));`
* `int register_netdev(struct net_device *device);` 注册网络设备
* `void unregitster_netdev(struct net_device *device);` 注销网络设备

## 打开和关闭 ##

驱动在加载入内核后，内核会调用probe函数来探测它。在网络接口可以传送数据包时，内核必须首先打开它并给它设置地址。内核打开和关闭网络接口是由ifconfig命令触发的。

* `int (*open)(struct net_device*);` 打开网络设备
* `int (*stop)(struct net_device*);` 关闭网络设备
* `void netif_start_queue(struct net_device*);` 启动网络传输队列
* `void netif_stop_queue(struct net_device*);` 关闭网络传输队列

## 网络数据的发送 ##

网络接口最重要的作用是发送和接收网络数据。

* 头文件：`<linux/skbuff.h>` 定义了网络驱动中传输的基本单元，`struct sk_buff`
* struct net_device_ops 网络设备驱动需要实现的接口函数
* `netdev_tx_t (*ndo_start_xmit) (struct sk_buff *skb,`
	   `struct net_device *dev);` 传输网络数据包的函数
* `void (*ndo_tx_timeout) (struct net_device *dev);` 传输超时函数

## 网络数据的接收 ##

接收网络数据相对于发送数据要复杂一些，因为你需要在原子上下文中把分配一个`sk_buff`并把它移交给上层处理。

数据包接收有两种实现方式：中断驱动和轮询。大多数驱动都是中断驱动的，有一些高吞吐量的驱动会使用轮询的方式。

* `struct sk_buff *dev_alloc_skb(unsigned int length);` 原子上下文中分配skb
* `printk_ratelimit()` 限制printk的输出频率，在中断相应函数中减少输出
* 实现高吞吐量的网络驱动，要减少网络阻塞最好的方法是使用napi，后面会介绍

## 中断处理 ##

硬件可以中断cpu发送两种事件：新的数据包到来和发送的数据包已经发送完成。

* 判断并处理数据包事件
* 如果在其他地方暂时停止了发送队列，应该在中断函数中重新启动它

## NAPI ##

高吞吐量的网络接口如果每个数据包都用中断来处理的话会给系统带来很大的负担，这个时候应该使用基于轮询的NAPI。这样可以减轻系统的负担，减少阻塞的时间。

只有极少数的设备实现了NAPI，因为实现起来比中断要复杂，而且有其他的一些条件。

在中断处理函数中，首先禁止进一步的中断处理，然后调度轮询函数，进入轮询函数后连续处理多个数据发送请求。

* `int (*poll)(struct net_device *dev, int *budget);` 网络驱动轮询函数
* `int netif_rx_schedule(struct net_device *dev);` 准备调用轮询函数
* `int (*poll)(struct net_device *dev, int *budget);` 轮询函数

## 其他 ##

* MAC Address Resolution
* Custom IOCTLs
* Multicasts

## 资源和链接 ##

* [网络设备驱动](http://lwn.net/images/pdf/LDD3/ch17.pdf)
* [网络设备驱动堆栈](http://www.ibm.com/developerworks/linux/library/l-linux-networking-stack/)
