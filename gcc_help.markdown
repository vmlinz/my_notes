# 使用GCC 帮助系统 #

前面在使用gcc 帮助的时候老是找不到自己想要的信息，而且手册巨长（有超过10^4页）。于是就RTFM找到一些使用gcc 帮助系统找到自己需要信息的方法，其实就是gcc 手册的帮助部分的内容。大家还有什么技巧都来讲讲，交流中学习。

## gcc --help ##

下面是从gcc mannual中摘录出来的

    Print (on the standard output) a description of the command line
    options understood by gcc.  If the -v option is also specified then
    --help will also be passed on to the various processes invoked by gcc,
    so that they can display the command line options they accept.  If the
    -Wextra option has also been specified (prior to the --help option),
    then command line options which have no documentation associated with
    them will also be displayed.

打印gcc 能识别的命令行选项（到标准输出）。如果同时指定了 -v 选项，那么 --help 选项会被传递给gcc 调用的所有工具，这样就能同时显示所有工具能识别的命令行选项了。如果指定 -Wextra 选项，那么没有相关文档的命令行选项也会显示出来。

gcc --help 会显示gcc 常用的选项和帮助，比如编译器的各个阶段

- -E 编译进行到预编译就停止，不继续汇编，编译或者链接
- -S 编译进行到生成汇编就停止，不编译和链接
- -c 编译成二进制对象就停止，不链接

## gcc --target-help ##

    Print (on the standard output) a description of target-specific command
    line options for each tool.  For some targets extra target-specific
    information may also be printed.

打印和目标平台相关的每个工具的命令行选项（到标准输出）。某些平台上会有额外的平台相关信息被打印出来。

## gcc --help={class|[^]qualifier}[,...] ##

    Print (on the standard output) a description of the command line
    options understood by the compiler that fit into all specified classes
    and qualifiers.  These are the supported classes:

打印编译器识别的属于某些分类或者修饰符的命令行选项到标准输出。

### 支持的分类 ###

- optimizers 优化器

	This will display all of the optimization options supported by the
	compiler.显示编译器支持的所有优化选项

- warnings 警告

	This will display all of the options controlling warning messages
	produced by the compiler.显示用于控制编译器产生警告信息的所有选项

- target 目标平台

	This will display target-specific options.  Unlike the
	--target-help option however, target-specific options of the linker
	and assembler will not be displayed.  This is because those tools
	do not currently support the extended --help= syntax.显示和目标平台相关
	的选项

- params 参数设置

	This will display the values recognized by the --param option.显示
	--param 选项接受的值

- language 语言

	This will display the options supported for language, where
	language is the name of one of the languages supported in this
	version of GCC.显示gcc 支持的程序语言选项

- common 公共选项

	This will display the options that are common to all languages.
	显示所有语言都支持的公共选项

### 支持的修饰符 ###

- undocumented 没有文档的

	Display only those options which are undocumented.
	显示没有文档的选项

- joined 联合选项

	Display options which take an argument that appears after an equal
	sign in the same continuous piece of text, such as: --help=target.
	显示联合选项，像 --help=target 这种

- separate 单独选项

	Display options which take an argument that appears as a separate
	word following the original option, such as: -o output-file.
	显示单独选项，想 -o output-file 这种

示例：

	--help=target,undocumented 获取目标平台没有文档的帮助文档
	--help=warnings,^joined,^undocumented 获取控制编译器警告的所有有文档的单
	独选项（^ 表示反选）

### 查询选项设置 ###

在帮助选项前面加上 `-Q` 选项后就可以查看当前的选项配置情况而不是得到选项的描述。

示例：

	gcc -Q -m64 --help=target
	查询x86_64目标平台的所有目标选项设置

	gcc -Q --help=c
	查询目标平台c语言的默认选项设置

	gcc -Q --help=warnings,joined
	查询警告控制的联合选项设置

## 资源 ##

man gcc
