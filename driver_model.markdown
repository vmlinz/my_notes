# Linux device driver model #
## driver binding ##

驱动绑定是联系驱动和相应设备的一个过程。一般由总线驱动来完成这个绑定，因为总线结构里面有表示特定总线的驱动和设备的结构。通过使用通用的设备和驱动结构，大多数的绑定都可以用通用的代码来实现。

### 总线 ###

总线结构体里面有一个该总线上所有设备的链表，每当有新的设备插入即调用device_\register的时候，设备就会被添加到链表末尾。它里面还有该总线类型的驱动链表，当调用driver\_register的时候，驱动就会被添加到驱动链表当中。这些就是触发驱动绑定的事件。

### device\_register ###

当添加新设备的时候，系统会遍历总线的驱动链表来查找支持该设备的驱动。为了确定这个，添加的ID必须是驱动支持的设备ID之一。这个匹配的函数是由总线驱动来提供的回调函数之一。如果匹配成功，该函数会返回1；失败则返回0。

int match(struct device * dev, struct device\_driver * drv);

如果能找到一个匹配的驱动，设备的驱动指针就被设置为该驱动，然后调用驱动的probe回调函数。这样驱动就可以进一步确定它是否支持该设备。

### Device Class ###

如果探测成功，设备就被注册到它所属的设备类型(device class)上。devclass\_add\_device函数被调用来在设备类型上枚举这个设备，然后通过设备类型的register\_dev回调把设备实际注册到设备类型上。

### Driver ###

当驱动被连接到设备之后，设备就被添加到驱动的设备链表上。

### sysfs ###

A symlink is created in the bus's 'devices' directory that points to
the device's directory in the physical hierarchy.
一个符号链接在总线的设备文件夹中创建，它指向设备文件夹中的实际设备项。

A symlink is created in the driver's 'devices' directory that points
to the device's directory in the physical hierarchy.
一个符号链接在驱动的设备文件夹中创建，它也指向设备文件夹中的实际设备项。

A directory for the device is created in the class's directory. A
symlink is created in that directory that points to the device's
physical location in the sysfs tree.
在类型文件夹下创建一个该设备的文件夹，该文件夹下会创建一个链接到该设备在sysfs树下的实际设备项。

A symlink can be created (though this isn't done yet) in the device's
physical directory to either its class directory, or the class's
top-level directory. One can also be created to point to its driver's
directory also.

### driver_register ###

The process is almost identical for when a new driver is added.
The bus's list of devices is iterated over to find a match. Devices
that already have a driver are skipped. All the devices are iterated
over, to bind as many devices as possible to the driver.
和添加驱动时的过程几乎相同，遍历总线的设备链表来找到合适的设备。已经匹配的设备被略过。所有的设备都会被遍历，并绑定尽可能多的设备到新加的驱动上。

### Removal ###

When a device is removed, the reference count for it will eventually
go to 0. When it does, the remove callback of the driver is called. It
is removed from the driver's list of devices and the reference count
of the driver is decremented. All symlinks between the two are removed.
设备卸载后，它的引用计数最后会减少到0。这个时候就会调用驱动的卸载回调函数。它从驱动的设备链表上删除，同时驱动的引用计数减1。驱动和设备之间的符号链接被删除。

When a driver is removed, the list of devices that it supports is
iterated over, and the driver's remove callback is called for each
one. The device is removed from that list and the symlinks removed.
当驱动卸载时，它所支持的设备链表将被遍历，同时对每个设备都执行驱动的卸载回调函数。设备从该链表上删除同时删除它们之间的符号链接。

## porting ##

In a nutshell, the driver model consists of a set of objects that can
be embedded in larger, bus-specific objects. Fields in these generic
objects can replace fields in the bus-specific objects.
概括的说，驱动模型中包含了一系列的对象，这些对象可以嵌入到更大的特定总线的对象。这些通用对象中的字段可以替换特定总线的相应字段。

驱动移植过程：

- Step 0: Read include/linux/device.h for object and function definitions.

- Step 1: Registering the bus driver. 使用bus\_register注册bus

- Step 2: Registering Devices.注册总线类型的设备
<br />.Initialize the device on registration.初始化通用设备。
<br />.Register the device.注册通用设备。

It is recommended that the generic device not be the first item in
the struct to discourage programmers from doing mindless casts
between the object types. Instead macros, or inline functions,
should be created to convert from the generic object type.
将通用的设备结构体嵌入到总线特定设备类型中，然后将设备注册到总线上。
最好不要把通用设备结构放在设备类型的第一个成员，避免无脑的强转设备类型。
应该使用宏或者inline函数来实现通用设备和特定设备类型的转换。

- Step 3: Registering Drivers.注册驱动

Embed a struct device_driver in the bus-specific driver.
Initialize the generic driver structure.
Register the driver.

- Step 4: Define Generic Methods for Drivers.定义通用的驱动方法
- Step 5: Support generic driver binding.提供通用的驱动绑定方法

The model assumes that a device or driver can be dynamically
registered with the bus at any time. When registration happens,
devices must be bound to a driver, or drivers must be bound to all
devices that it supports.
驱动模型假定设备或者驱动可以在任意时刻被动态地注册到总线上。注册的时候，所有的设备必须绑定到一个驱动上，或者驱动必须绑定到所有它支持的设备上。

  int (*match)(struct device * dev, struct device_driver * drv);

- Step 6: Supply a hotplug callback.提供热插拔回调
<br />.ACTION: set to 'add' or 'remove'
<br />.DEVPATH: set to the device's physical path in sysfs.

The driver model core passes several arguments to userspace via
environment variables.

- Step 7: Cleaning up the bus driver.清理总线驱动
<br />Device list.
int bus\_for\_each\_dev(struct bus\_type * bus, struct device * start, void * data, int (*fn)(struct device *, void *));
<br />Driver list.
int bus\_for\_each\_drv(struct bus_type * bus, struct device\_driver * start, void * data, int (*fn)(struct device\_driver *, void *));
