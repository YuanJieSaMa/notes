# SDCC

首先我们要下载**交叉编译工具** *SDCC*，用包管理器和官网包都可以.

以Arch为例：

```shell
yay -S SDCC
```



# Stcflash

接着要安装**烧写工具** *Stcflash*.

我们可以直接上[Github下载](https://github.com/laborer/stcflash).

解压后我们需要给*stcflash.py*运行权限，并添加至**环境变量**

```shell
chmod -ax stcflash.py
```



```shell
sudo mv stcflash.py /usr/local/bin/
```

接着我们需要一个Python的**串口驱动**.

```shell
yay -S python-pyserial
```

# 编译

* 值得注意的是

在SDCC环境下，我们的8051单片机头文件为

```c
#include <mcs51/8051.h>
```

在自己的编译器下写好了C程序后，使用SDCC编译：

```shell
sdcc $file.c$ -o $file.ihx$
```

* SDCC生成的是ihx文件，但是51单片机烧写是要.hex文件，所以这里我们需要使用packihx命令（SDCC自带）

```shell
packihx $file.ihx$
packihx $file.ihx$ > $file.hex$
```

# 烧录

使用之前安装的*stcflash*

```shell
sudo stcflash.py $file.hex$
```



如果自动检测的串口不正确，我们可以使用**指定窗口**

```shell
sudo stcflash.py $file.hex$ --port /dev/ttyUSB0
```



