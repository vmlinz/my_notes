# Linux device driver model #

## overview ##

linux 设备模型的目的是为内核构建统一的模型，从而使系统有一般性的描述。

设备模型的几个主要任务:

* 电源管理和系统关机：设备模型使得操作系统能够以正确的顺序遍历关闭系统硬件
* 和用户空间进行通讯：sysfs虚拟文件系统的实现和设备模型密切相关，并向用户空间展现了系统的结构
* 设备模型：维护设备驱动的体系化数据机构
* 对象的生命周期：设备模型的创建需要一系列的机制来处理对象的生命周期、对象之间的关系以及这些对象在用户空间的表示

## kobject & kset ##

* A kobject is an object of type struct kobject.  Kobjects have a name
  and a reference count.  A kobject also has a parent pointer (allowing
  objects to be arranged into hierarchies), a specific type, and,
  usually, a representation in the sysfs virtual filesystem.
  kobject是kobject结构的对象。kobject对象有名字和引用计数。
* A ktype is the type of object that embeds a kobject.  Every structure
  that embeds a kobject needs a corresponding ktype.  The ktype controls
  what happens to the kobject when it is created and destroyed.
  ktype是嵌入有kobject的对象的类型。每个嵌入了kobject的结构都需要一个相应的ktype类
  型。ktype主要负责管理对象的创建和销毁。
* A kset is a group of kobjects.  These kobjects can be of the same ktype
  or belong to different ktypes.  The kset is the basic container type for
  collections of kobjects. Ksets contain their own kobjects, but you can
  safely ignore that implementation detail as the kset core code handles
  this kobject automatically.
  kset是一系列kobject对象的集合。这些对象可以是同一个ktype也可以属于不同的ktype。
  kset是kobject对象的基本容器。kset自己本身也有自己的kobject对象。

### Embedding kobjects  ###

和内核链表一样，kobject很少被直接使用，一般都是嵌入其他结构。可以认为它是对象的基类。例如：
`
struct uio\_map {
	struct kobject kobj;
	struct uio_mem *mem;
};
`

要取得嵌入其他结构体中的kobject，只需要直接取对应的成员变量即可；要通过kboject获取它所在的结构体则需要使用container_of宏。

### kobjects的操作接口 ###

`void kobject_init(struct kobject *kobj, struct kobj_type *ktype);`

`int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...);
`

`int kobject_rename(struct kobject *kobj, const char *new_name);
`

`const char *kobject_name(const struct kobject * kobj);
`

`int kobject_init_and_add(struct kobject *kobj, struct kobj_type *ktype,
			     struct kobject *parent, const char *fmt, ...);
`
### Uevents ###

After a kobject has been registered with the kobject core, you need to
announce to the world that it has been created.  This can be done with a
call to kobject_uevent():
kobject在核心中注册之后，你需要告知系统它被创建了。可以通过调用kobject\_uevent()来完成。

`int kobject_uevent(struct kobject *kobj, enum kobject_action action);`

### Reference counts ###

One of the key functions of a kobject is to serve as a reference counter
for the object in which it is embedded. As long as references to the object
exist, the object (and the code which supports it) must continue to exist.
The low-level functions for manipulating a kobject's reference counts are:
kobject的一个主要功能之一就是作为嵌入对象的引用计数器。主要的操作方法如下：

`struct kobject *kobject_get(struct kobject *kobj);`

`void kobject_put(struct kobject *kobj);`

Because kobjects are dynamic, they must not be declared statically or on
the stack, but instead, always allocated dynamically.
由于kobject对象是一个动态数据结构，所以它们不能被声明为静态或者在栈上分配，它们只能动态分配在堆上。

### ktypes and release methods ###

struct kobj_type结构中有kobject的release函数，用于销毁kobject。

### ksets ###

A kset is merely a collection of kobjects that want to be associated with
each other.  There is no restriction that they be of the same ktype, but be
very careful if they are not.

kset只是一系列有关联的kobject集合。它们不必是同一个ktype，但是如果它们不是同一个ktype则需要非常仔细的处理它们。

kset的主要功能：
- It serves as a bag containing a group of objects. A kset can be used by
  the kernel to track "all block devices" or "all PCI device drivers."

- A kset is also a subdirectory in sysfs, where the associated kobjects
  with the kset can show up.  Every kset contains a kobject which can be
  set up to be the parent of other kobjects; the top-level directories of
  the sysfs hierarchy are constructed in this way.

- Ksets can support the "hotplugging" of kobjects and influence how
  uevent events are reported to user space.支持热插拔，同时影响uevent如何被报告到用户空间。

接口：

`struct kset *kset_create_and_add(const char *name,
				   struct kset_uevent_ops *u,
				   struct kobject *parent);
`

`void kset_unregister(struct kset *kset);
`

If a kset wishes to control the uevent operations of the kobjects
associated with it, it can use the struct kset_uevent_ops to handle it:
通过uevent操作函数来控制和它相关的kobject。

`struct kset_uevent_ops {
	int (*filter)(struct kset *kset, struct kobject *kobj);
	const char *(*name)(struct kset *kset, struct kobject *kobj);
	int (*uevent)(struct kset *kset, struct kobject *kobj,
		      struct kobj_uevent_env *env);
};
`

### Kobject removal ###

After a kobject has been registered with the kobject core successfully, it
must be cleaned up when the code is finished with it.  To do that, call
kobject\_put().  By doing this, the kobject core will automatically clean up
all of the memory allocated by this kobject.  If a KOBJ\_ADD uevent has been
sent for the object, a corresponding KOBJ\_REMOVE uevent will be sent, and
any other sysfs housekeeping will be handled for the caller properly.
kobject成功注册到核心之后，在使用它的代码结束后它必须被清理。通过调用kobject\_put()来实现。这样核心就会自动清除给这个kobject分配的内存。如果之前发送过KOBJ\_ADD事件，此时会发送相应的KOBJ\_REMOVE事件。

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

### driver\_register ###

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

Embed a struct device\_driver in the bus-specific driver.
Initialize the generic driver structure.
Register the driver.

- Step 4: Define Generic Methods for Drivers.定义通用的驱动方法
- Step 5: Support generic driver binding.提供通用的驱动绑定方法

The model assumes that a device or driver can be dynamically
registered with the bus at any time. When registration happens,
devices must be bound to a driver, or drivers must be bound to all
devices that it supports.
驱动模型假定设备或者驱动可以在任意时刻被动态地注册到总线上。注册的时候，所有的设备必须绑定到一个驱动上，或者驱动必须绑定到所有它支持的设备上。

  int (*match)(struct device * dev, struct device\_driver * drv);

- Step 6: Supply a hotplug callback.提供热插拔回调
<br />.ACTION: set to 'add' or 'remove'
<br />.DEVPATH: set to the device's physical path in sysfs.

The driver model core passes several arguments to userspace via
environment variables.

- Step 7: Cleaning up the bus driver.清理总线驱动
<br />Device list.
int bus\_for\_each\_dev(struct bus\_type * bus, struct device * start, void * data, int (*fn)(struct device *, void *));
<br />Driver list.
int bus\_for\_each\_drv(struct bus\_type * bus, struct device\_driver * start, void * data, int (*fn)(struct device\_driver *, void *));

## Platform Devices and Drivers ##

See <linux/platform\_device.h> for the driver model interface to the
platform bus:  platform\_device, and platform\_driver.  This pseudo-bus
is used to connect devices on busses with minimal infrastructure,
like those used to integrate peripherals on many system-on-chip
processors, or some "legacy" PC interconnects; as opposed to large
formally specified ones like PCI or USB.
平台总线是一个虚拟总线，它被用于链接集成在SOC处理器总线上的设备。

### Platform devices ###

Platform devices are devices that typically appear as autonomous
entities in the system. This includes legacy port-based devices and
host bridges to peripheral buses, and most controllers integrated
into system-on-chip platforms.  What they usually have in common
is direct addressing from a CPU bus.  Rarely, a platform\_device will
be connected through a segment of some other kind of bus; but its
registers will still be directly addressable.
平台设备的一个共性就是他们都可以通过cpu总线直接寻址访问；即使偶尔通过其他类型的总线访问，此时设备的寄存器都是可以被直接寻址的。

### Platform drivers ###
Platform drivers follow the standard driver model convention, where
discovery/enumeration is handled outside the drivers, and drivers
provide probe() and remove() methods.  They support power management
and shutdown notifications using the standard conventions.
平台设备遵从标准的驱动模型惯例，驱动的发现和枚举发生在驱动程序以外，驱动负责提供probe()和remove()等方法。

Platform drivers register themselves the normal way:
- int platform\_driver\_register(struct platform\_driver *drv);

### Device Enumeration ###
As a rule, platform specific (and often board-specific) setup code will
register platform devices:
特定平台(特定板卡)的设置代码中注册平台设备是一个规则。

.int platform\_device\_register(struct platform\_device *pdev);
.int platform\_add\_devices(struct platform\_device **pdevs, int ndev);

The general rule is to register only those devices that actually exist,
but in some cases extra devices might be registered.  For example, a kernel
might be configured to work with an external network adapter that might not
be populated on all boards, or likewise to work with an integrated controller
that some boards might not hook up to any peripherals.

In some cases, boot firmware will export tables describing the devices
that are populated on a given board.   Without such tables, often the
only way for system setup code to set up the correct devices is to build
a kernel for a specific target board.
一些情况下，启动固件会导出给定开发板上提供的设备描述表。如果没有这种设备描述表，系统正确设置设备的唯一方法就是构建一个目标平台专用的内核。这种内核在嵌入式开发中比较常见。

Driver binding is performed automatically by the driver core, invoking
driver probe() after finding a match between device and driver.  If the
probe() succeeds, the driver and device are bound as usual.
驱动绑定是由驱动核心自动执行的，通过发现匹配的设备和驱动后调用驱动的probe()函数。如果probe()成功，驱动和设备就绑定起来了。
