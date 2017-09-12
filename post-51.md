使用 QEMU 模拟 PowerPC 版本的 Mac OS X / Classic Mac OS
===
目录：

[TOC]

首先要说声抱歉，由于我已经升入高中并且主打文化课，博文可能很难保持长期的更新。这篇博文涉及的东西早在一周前就已经做完，但一直拖着直到今天才开始写。话不多说，进入主题吧。

## 1. 准备镜像
我们要模拟 PowerPC 版本的 Mac OS X，所以要准备好对应版本的 Mac OS X 的镜像。我发现有一个叫做 [MacintoshGarden.org](http://macintoshgarden.org/) 的网站，收集了许多旧版本 Mac OS X / Classic Mac OS 的资源。大家可以先下载着。

## 2. QEMU 版本下载
模拟 Mac OS X 10.4 之前的版本，只需要下载标准版本的  qemu-system-ppc 即可。只需要通过 brew 安装即可。这里给出 Emaculation.com 提供的 QEMU 的 [Windows 版本](http://www.emaculation.com/forum/viewtopic.php?f=34&t=9028)和 [macOS 版本](http://www.emaculation.com/forum/viewtopic.php?f=34&t=8848)。只需要安装置顶的版本即可。qemu.command 可配置如下 (Windows 用户请根据实际使用 Shell 情况修改):
```bash
#!/bin/bash
cd "$(dirname "$0")"
./qemu-system-ppc -hda osx_cheetah.qcow2 -hdb swap.img -cdrom cheetah.iso -m 512 -boot c -net nic,model=e1000 -net user
```
其中，-hda、-hdb 设置的是内置硬盘的镜像，-cdrom 是设置光盘镜像的路径。-m 是 RAM 的大小(以 MB 为单位)。而 -boot 则是第一启动项设置，c 为硬盘，d 为光盘。后面的 -net 是指网络配置，无需修改。  
而对于 Mac OS X 10.4 以及以上版本，我们需要一个特殊版本的支持 mac99p 机型的 QEMU，由于我使用的是 macOS，故无法提供 Windows 版本。Mac 版本的下载我已经上传 CDN 了。下载地址在[这里](https://static.chickger.pw/201709/qemu-mac99p.zip)。
这个版本我没有内置 qemu.command，需要进行如下配置:
```bash
#!/bin/bash
cd "$(dirname "$0")"
./qemu-system-ppc-wip -bios openbios-qemu-wip.elf -L pc-bios -boot c -m 1024 -M mac99p -prom-env "auto-boot?=true" -net nic,model=e1000 -net user -hda osx_tiger.qcow2 -hdb swap.img
```
其中，-bios 是设置特殊的 BIOS 文件。而 -M 则是设置机型，Mac OS X Tiger 要求至少是 G4 的 Mac，所以机型设置为 mac99p。而 -prom-env 则是在 NVRAM 中写入参数，比如如果你需要 -v 啰嗦启动，则加入 `-prom-env "boot-args=-v"`。

## 3. 虚拟硬盘镜像生成
QEMU 需要你手动生成虚拟硬盘镜像，这里我们使用 QEMU 发布的实用工具生成镜像。  
其中，虚拟硬盘镜像格式很多，常用的有 RAW、qcow2 等。如果要生成一个大小为 10 GB 的 RAW 镜像，则执行命令为:
```bash
qemu-img -f raw -O size=10G Filename.img
```
RAW 镜像的优点在于 macOS 可以直接挂载(如果文件系统受支持)。缺点是占空间大，即使是空白镜像也要写入所有的 0。空间不能动态调节。  
而 qcow2 镜像的优点在于按需占用空间，而不是一次性分配。在没有写入内容时只占用很小的空间，而占用空间的大小取决于你装入的数据的多少。缺点是 macOS 不能直接识别这种格式，挂载不是很方便。如果生成一个 10 GB 的 qcow2 镜像，则执行命令为:
```bash
qemu-img -f qcow2 -O size=10G Filename.qcow2
```
## 4. 安装操作系统
上面全部配置完成，就可以安装了。这个由于涉及到跨架构模拟，性能损失可能比较严重。因此，请务必耐心充足。  
你可以看看 QEMU 的帮助文档，发现更多有用、有趣的配置项，让你的 Mac OS 更加出彩。
下面贴一张 Mac OS X v10.0 "Cheetah"  的运行图片:

![Mac OS X Cheetah](https://static.chickger.pw/201709/osx10.0/First%20Boot%2017.png)