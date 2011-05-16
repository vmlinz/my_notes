# 0.Driver basics #

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

- obj-y build builtin object for the kernel
- obj-m build loadable module for the kernel, which will be used later
- <module_name>-y multiple object to be built into one module

#### build external kernel modules ####

- command to build kernel module

	make -C <path_to_kernel_src> M=$PWD modules

- build for running kernel

	make -C /lib/modules/$(shell uname -r)/build M=$PWD modules

## 3.running a module ##

The kbuild system specifies how to build a linux

- insmod

	install kernel modules into kernel memory

## 4.the very "hello world" module ##

See my source code and run it
