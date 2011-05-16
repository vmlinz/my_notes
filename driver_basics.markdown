# Driver basics #

notes on linux kernel driver basics

## development environment ##

linux kernel development environment

- building tools: gcc, make, modutils etc.

- download linux kernel source tree:

	git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2
	.6.git

## building a module ##

process to build a linux kernel module

### prerequisites ###

- build tools
- kernel source tree
- build for your running kernel

### intro to kbuild system ###

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

## running a module ##

The kbuild system specifies how to build a linux

- insmod

	install kernel modules into kernel memory

## the very "hello world" module ##

See my source code and run it
