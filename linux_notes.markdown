# Install Linux From Livecd Iso #

grub2启动菜单默认是隐藏的，除非您改动了/etc/default/grub中的设置。在开机启动时一直按Shift键，直到grub2启动菜单出现，这时候可以按上下方向键选择一个项目

在/etc/grub.d/40_custom（系统自带）中进行编辑，或者自行新建50_ubuntu进行编辑，应该注意的是，这些文件必须是可执行的，才能够由update-grub写入二进制文件，因此必须赋予可执行权限：

* sudo chmod +x /etc/grub.d/40_custom

* 编辑40_custom

`menuentry "Ubuntu 10.04 LiveCD" {
set root='(hd1,1)'
loopback loop (hd1,1)/ubuntu-10.04-desktop-i386.iso
linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=/ubuntu-10.04-desktop-i386.iso ro quiet splash locale=zh_CN.UTF-8
initrd (loop)/casper/initrd.lz
boot
}
`

其中，menuentry "xxx" { }为固定语法，必须要写。注意，{ }内的行与行之间不能有空行，必须是连续的。 loopback为grub2的新增功能，用于载入镜像文件。 grub2中，kernel命令已经被替换为linux；root已经被替换为set root hdx,x。


* NOTE:
安装时要特别注意 执行下面的命令
sudo umount -l /isodevice
