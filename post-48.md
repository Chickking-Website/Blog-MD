让旧的 iPhone 成为服务器
===
目录:

[TOC]

最近家里的一台服役了 13 年的老笔记本退役了，本来是作为家里校园网 Dr.COM 验证的专用机器 + 内网 Web 服务器的。对于笔记本运行发出的噪音的不满，我决定不使用笔记本电脑或台式机来接替它，而是使用 iPhone。  
## 0. Dr.COM 验证程序
### a. 程序本身的配置
最近闲逛 GitHub 时发现了一个可以直接用的 Dr.COM 认证的 Python 项目，而且也可以配合路由器使用，叫做 [drcom_generic](https://github.com/drcoms/drcom-generic)。  
**<span style="color: red;">注: 建议大家直接去看该项目的 wiki，这里的 Dr.COM 配置方法不一定适合每一个校园网的网络环境。</span>**  
首先我们 Clone 下来这个项目，然后我们不需要别的，先用 Wireshark 抓一下原版客户端登录的包，保存为 dump.pcapang。  
然后打开 Auto Configure 工具，上传 pcapang 包，即可生成你独有的配置文件。  
Example:
```
server = '192.168.88.66'
username = '123456'
password = 'pass000'
CONTROLCHECKSTATUS = '\x20'
ADAPTERNUM = '\x01'
host_ip = '127.0.0.1'
IPDOG = '\x01'
host_name = 'foo'
PRIMARY_DNS = '192.168.0.1'
dhcp_server = '0.0.0.0'
AUTH_VERSION = '\x0f\x00'
mac = 0x00aabbccddee
host_os = 'Windows 10'
KEEP_ALIVE_VERSION = '\xdc\x02'
ror_version = False
```
这时候我们把项目文件夹下的 latest-wired.py 拷贝出来，并命名为 Drcom.py。  
找到 `# CONFIG` 和 `# CONFIG_END` 之间的部分，把这一部分换成你自己的配置。(截止到2017年4月25日，这一部分是第13行到第32行)  
如果这样就运行，会产生 log 文件，长期这样这个文件会很大，我们可以设置 LOG_PATH 来规避这个问题。找到整段代码中两处 LOG_PATH 的值(第81行~第84行)，并设置为:
```python
LOG_PATH = '/dev/null'
if IS_TEST:
    DEBUG = True
    LOG_PATH = '/dev/null'
```
注: 如果是用 Windows 的同学想在电脑上跑这个 Python 脚本，可以把 LOG_PATH 设置为 `NUL`。  
这样文件本身的修改配置就做完了。
### b. iPhone 环境配置
首先我们需要一台越狱的 iPhone，版本任意。
在 Cydia 里面先安装上 `MobileTerminal`、`MobileTerm Backgrounder`、`Core Utilities`等。  
Cydia/Telesphoreo 源里面的 Python 不要装，版本比较老，我发现 Linus Yang 大神写了一个 Python 2.7.6 的 deb 插件，可以去 GitHub 上[围观](https://github.com/linusyang/python-for-ios)一下。我这里也提供直连的[下载地址](https://github.com/linusyang/python-for-ios/releases/download/v2.7.6-3/python_2.7.6-3_iphoneos-arm.deb)。  
安装好 Python，就可以把脚本放在这上面跑了。需要注意的是，作为服务器，应当尽可能的使其实现最大化的自动化，而这个脚本一旦断线是不会自动重连的。所以我们可以写一个  Shell 死循环，使其重复执行。  
这里有同学会提问题: 那么如果我们想关掉它怎么办，`killall bash && killall python` 会把我们当前操作的终端也一起关闭啊？这个问题可以简单粗暴地解决，即建立 bash 的软链接，取另一个名字，然后让其解释执行我们的脚本。代码如下:
```bash
#!/bin/abcsh

for((i=1;i<=10;i=i));
do
python /var/mobile/Drcom.py
done
```
可以看出，我们建立了一个 bash 的软链接为 `/bin/abcsh`，这样当我们想要关闭 Dr.COM 验证程序时只需要一行 `killall abcsh && killall python` 即可。  
## 1. Web Server 的配置
iPhone 上运行 Web Server，最省资源的方式就是使用 lighttpd。

lighttpd 的配置方式非常简单，首先我们从 Cydia 中下载这个插件，然后在终端启动即可，不多说了，直接上配置。
```
server.document-root = "/var/mobile/Documents/www" 

server.port = 80

server.username = "mobile"

mimetype.assign = (
  ".html" => "text/html; charset=utf-8",
  ".htm" => "text/html; charset=utf-8",
  ".css" => "text/css; charset=utf-8",
  ".js" => "text/javascript; charset=utf-8",
  ".txt" => "text/plain; charset=utf-8",
  ".jpg" => "image/jpeg",
  ".png" => "image/png",
  "" => "application/octet-stream; charset=utf-8"
)

dir-listing.activate = "enable" 

dir-listing.encoding = "utf-8"

index-file.names = ( "index.html" )
```
将配置文件保存到 /etc/lighttpd.conf。
## 2. 维护相关
iPhone 服务器相对来说较容易维护，我使用 Veency + OpenSSH 来维护 iPhone 服务器。  
Veency 是 iPhone 上的 VNC Server，可以让你通过 VNC 来控制 iPhone，大家下载了就知道。  
给大家看看 Veency 的效果:
![Veency on iPhone](https://static.chickger.pw/201704/iPhoneVeency.png)
OpenSSH 不必多说，诸位老司机都懂，但切记要把 root 密码 alpine 改掉，不然你的 iPhone 就不是你的了。  
vsftpd 也有必要，这主要用于 iPhone Web Server 上资源下载目录的上传操作，直接上配置吧。路径: /etc/vsftpd.conf
```
local_root=/var/mobile/Documents/down/
anon_root= /usr/share/empty
listen=YES
background=YES
local_enable=YES
write_enable=YES
ftp_username=upload
ftp_password=upload
```
## 3. 结语
iPhone 作为服务器还是十分给力的，首先配置相对大多数服务器要高，而且功耗小，运行无声音，发热也小，很适合扮演家庭服务器的角色。