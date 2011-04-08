轻量级Emacs--JED
==========

# JED介绍 #

JED是一个轻量级的Emacs实现，它是JohnE.Davis在1992年发起的一个编辑器项目。   
这个编辑器现在仍然在活跃的开发中，主要开发者仍然是最初的作者。 PS:他最近也   
使用git来管理这个项目了。

JED通过S-lang（作者自己写的一个语法类似于c语言的解释器）来扩展，它的设计   
结构和GNU Emacs是基本一致的（核心+解释器+扩展）。但是它非常的轻量级，同   
时具有命令行界面和x界面。

# 为什么使用JED #

GNU Emacs是我最喜欢的编辑器，但是有时候需要在终端下面编辑一些配置文件。   
在vps上使用的时候就不太方便，需要重新配置。而且它太重量级了。曾经有段时间   
我都把EDITOR变量设置成vim了，但是在两个足够复杂的编辑器之间切换，我的手   
指不听话。我也用emacs-daemon，但是有时候emacsclient不能连接上已有的   
emacs-server，结果老是重新启动一个新的实例。

我也使用过nano、zile和micro emacs这些轻量级编辑器，但是始终不给力。nano   
的按键绑定不是类Emacs的；zile够轻量级，但是不支持UTF，连基本的高亮显示   
都不支持。jasspa的Micro Emacs，它对UTF的支持也不完善，甚至到现在也只有   
少数Linux发行版（gentoo）给它打包。

终于我找到了一个功能强大，同时完美支持UTF-8，也非常轻量的Emacs实现。它的   
扩展语言也非常易用，使用接近C语言的语法，让我觉得非常亲切。这样我就不用   
在做基本编辑的时候需要去使用其他和emacs按键差别很大的编辑器了。我相信喜欢   
GNU Emacs的同学一定会爱上这个轻量级的emacs。

# JED的特性 #

1. 语法高亮
2. 易用的下拉菜单UI
3. 各种主流编辑器的按键绑定模拟（Emacs,EDT,CUA...）
4. 非常丰富的编程语言模式支持
5. TeX支持
6. 终端鼠标支持（GPM）
7. 和GNU Emacs几乎一样的块操作
8. 异步子进程支持
9. UTF-8支持
10. 多种平台支持

# 安装和定制JED #

JED默认使用GNU Emacs的按键绑定，非常符合我的胃口。我现在使用的是Ubuntu，   
下面是我安装和定制JED的简单介绍。

1. 安装JED：`sudo aptitude install jed jed-extra`
2. 复制Debian包的配置文件：
`zcat /usr/share/doc/jed/examples/jed.rc.gz > ~/.jedrc`
3. 编辑~/.jedrc：`jed ~/.jedrc`
4. 禁止显示菜单，这样就和我的Emacs配置一样了：`enable_top_status_line(0);`
5. 打开自动解压，这样就可以直接浏览压缩文本：`auto_compression_mode(1);`
6. 设置C-mode的缩进风格：`c_set_style ("linux");`
7. 其他的模式以及进一步的定制请参考
[JED参考](http://www.jedsoft.org/jed/doc/jedfuns.html)

# JED文档和资源 #

1. JED 官方网站:[http://www.jedsoft.org/jed](http://www.jedsoft.org/jed)
2. JED 扩展列表:
[http://jedmodes.sourceforge.net/modes](http://jedmodes.sourceforge.net/modes)
3. JED的debian package链接:
[http://packages.debian.org/squeeze/jed](http://packages.debian.org/squeeze/jed)
