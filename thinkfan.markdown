# 使用thinkfan智能控制thinkpad风扇速度 #

我的thinkpad在安装了ubuntu 10.04之后，cpu风扇一直是满速运转，风扇声音非常烦人，而
<br />且长时间这样运转可能会把cpu风扇给搞坏了。于是开始在网上找控制thinkpad风扇的方
<br />法。这里就简单记录一下使用thinkfan来控制cpu风扇的过程。

<!--more-->

## 安装thinkfan ##

thinkfan的配置非常简单，它通过读取配置文件(/etc/thinkfan.conf)中的(FAN\_LEVEL, LOWER\_LIMIT, UPPER\_LIMIT)
<br />三元组来实现使用温度上下限来控制风扇转速级别。底层是需要有thinkpad_acpi内核模块的支持，
<br />内核模块通过/proc/acpi/ibm/fan这个procfs节点给用户空间提供了风扇控制的具体方法。

### 打开thingkpad_acpi的风扇控制 ###

首先要打开thinpad_acpi的风扇控制支持，具体做法是在加载这个内核模块的时候给它传递一
<br />个打开风扇控制的变量。具体做法如下：

`modprobe thinkpad_acpi fan_control=1 experimental=1`

要实现开机加载模块时设置，则需要在/etc/modprobe.d/下增加一个配置文件thinkpad-acpi.conf，内容：

`options thinkpad_acpi experimental=1 fan_control=1`

### 安装thinkfan ###

安装ubuntu软件仓库里面的thinkfan和sysfsutils(读取和设置sysfs属性)。

`aptitude install thinkfan sysfsutils`

thinkfan会在系统的运行级别中添加thinkfan的后台服务，这样就可实现开机运行。

### 配置和激活thinkfan ###

阅读/etc/thinkfan.conf可以知道它是通过thinkpad_acpi提供的风扇控制接口来实现风扇控制的，
<br />它给风扇的转速级别对应了一个上下限温度，在这个上下限内就设置对应的转速。

安装后发现thinkfan并没有直接生效，阅读了/etc/init.d/thinkfan脚本后发现还需要设置/etc/default/thinkfan
<br />文件中的START=yes。

到这里，thinkpad的风扇控制就应该打开了。

## 玩thinkpad_acpi的风扇控制接口 ##

cat /proc/acpi/ibm/fan可以得到控制风扇的方法。

* echo 'level 0' | sudo tee /proc/acpi/ibm/fan (fan off)
* echo 'level 2' | sudo tee /proc/acpi/ibm/fan (low speed)
* echo 'level 4' | sudo tee /proc/acpi/ibm/fan (medium speed)
* echo 'level 7' | sudo tee  /proc/acpi/ibm/fan (maximum speed)
* echo 'level auto' | sudo tee /proc/acpi/ibm/fan (automatic - default)
* echo 'level disengaged' | sudo tee /proc/acpi/ibm/fan (disengaged)

我们可以看到其中有一个auto选项，说明驱动本身是要自己实现风扇只能控制的，我之前查看>
<br />的时候就发现驱动的默认设置就是auto，可惜它么能正常工作。

## 资源和链接 ##

* [thinkpad风扇控制脚本](http://www.thinkwiki.org/wiki/Fan_control_scripts)
* [如何控制风扇](http://www.thinkwiki.org/wiki/Patch_for_controlling_fan_speed)
* [硬件温度传感器](http://www.thinkwiki.org/wiki/Thermal_sensors)
