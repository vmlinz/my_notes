# 0.Driver basics -- kernel module programming #

notes on linux kernel driver basics

## 0.what's kernel module ##

	What exactly is a kernel module? Modules are pieces of code that can be
	loaded and unloaded into the kernel upon demand. They extend the functio
	nality of the kernel without the need to reboot the system. For example,
	one type of module is the device driver, which allows the kernel to acce
	ss hardware connected to the system. Without modules, we would have to b
	uild monolithic kernels and add new functionality directly into the kern
	el image. Besides having larger kernels, this has the disadvantage of re
	quiring us to rebuild and reboot the kernel every time we want new funct
	ionality.

## 1.development environment ##

linux kernel development environment

- 1.0.building tools: gcc, make, modutils etc.

- 1.1.download linux kernel source tree:

	git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2
	.6.git

## 2.building a module ##

process to build a linux kernel module

### 2.0.prerequisites ###

- build tools
- kernel source tree
- build for your running kernel

### 2.1.intro to kbuild system ###

Linux uses kbuild system to build its source code. The kbuild system simplifies the build process. Refer `Documentation/kbuild` to get more information.

#### builtin targets and variables ####

- obj-y builds builtin object for the kernel
- obj-m builds loadable module for the kernel, which will be used later
- `<module_name>-y` multiple object to be built into one module

#### build external kernel modules ####

- command to build kernel module

	make -C <path_to_kernel_src> M=$PWD modules

- build for running kernel

	make -C /lib/modules/$(shell uname -r)/build M=$PWD modules

## 3.running a module ##

The kbuild system specifies how to build a linux, and here are mod tools which installs and uninstalls a specific kernel module.(more on module−init−tools)

- insmod

	install kernel modules into kernel memory

- rmmod

	uninstall loaded kernel module from kernel memory

## 4.the very "hello world" module ##

See my source code and run it

- module information(licensing and documentation)
- module init and exit functions
- module params
- kbuild system
- build external kernel module
- build multiple modules

## 5.rational ##

How kernel module begins - module_init

	A module always begin with eitherthe init_module or the function
	you specify with module_init call. This is the entry function for
	modules; it tells the kernel what functionality the module provides
	and sets up the kernel to run the module's functions when
	they're needed.

How it ends - module_exit

	All modules end by calling either cleanup_module or the function you spe
	cify with the module_exit call. This is the exit function for modules; i
	tundoes whatever entry function did. It unregisters thefunctionality
	that the entry function registered.

Available functions - printk and other kernel functions

	The definition for the symbols comes from the kernel itself; the only
	external functions you can use are the ones provided by the kernel.
	see /proc/kallsyms

How printf runs - strace and system calls

	Library functions are higher level, run completely in user space and pro
	vide a more convenient interface for the programmer to the functions tha
	t do the real work−−−system call


# 1.Driver basics -- types of drivers #

notes on types of device drivers, demonstrated with char device drivers(the most common device driver in linux kernel)

## 1.Char Device Driver ##

### 1.0.struct file_operations ###

	Each device is represented in the kernel by a file structure, which is d
	efined in linux/fs.h. Generic file operations are redirected to device s
	pesific file_operations, which implement the real operations.

struct file_operations
- open
- read
- write
- ioctl(`unlocked_ioctl, compat_ioctl`)
- release

### 1.1.register char device ###

`int register_chrdev(unsigned int major, const char *name, struct file_operations *fops);`

	where unsigned int major is the major number you want to request, const
	char *name is thename of the device as it'll appear in /proc/devices and
	struct file_operations *fops is apointer to the file_operations table fo
	r your driver.

kernel use major number to identify a driver for a type of device, minor number is used for identify a specified device

### 1.2.unregister char device ###

	We can't allow the kernel module to be rmmod'ed whenever root feels like
	it. If the device file is opened by a process and then we remove the ker
	nel module, using the file would cause a call to the memory location whe
	re the appropriate function (read/write) used to be. So we ref count on
	the module.

- `try_module_get(THIS_MODULE): Increment the use count`
- `module_put(THIS_MODULE): Decrement the use count`
- `int unregister_chrdev(unsigned in major, const char *name);`

### 1.3.(really)simple char device driver ###

- register char device
- unregister char device
- implement fields of struct file_operations(open, release and read)
- module ref count and module cleanup
