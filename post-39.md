<!-- 【Hack】ASUS Pro451L 黑苹果 98% 完美 -->
哈哈，终于到了寒假，又可以浪一番了。这一阵子搞黑苹果倒是搞得很欢。话不多说，刚刚把老妈的本子 98% 完美了。放一下我都干了啥吧。
## 1. 机器介绍
#### a) 配置单
`` // 打钩项目为已驱动的项目。``

CPU: Intel Core i5-4200U 2.39GHz <input type="checkbox" checked disabled /></input>

内存: 4GB DDR3 1600MHz <input type="checkbox" checked disabled /></input>

硬盘: 日立 7200 转 500GB <input type="checkbox" checked disabled /></input>

显卡: Intel HD Graphics 4400 1536MB <input type="checkbox" checked disabled /></input> + NVIDIA GeForce 820M 2GB <input type="checkbox"  disabled ></input> (Optimus 技术独显无法使用)

声卡: Conexant SmartAudio CX20751/2 <input type="checkbox" checked disabled ></input> (麦克风不可用)

网卡: Realtek RTL8168 + Broadcom BCM94352HMB(更换) <input type="checkbox" checked disabled /></input>

蓝牙: Broadcom BCM94352HMB <input type="checkbox" checked disabled /></input>

系统版本： OS X El Capitan 10.11.6 (15G31)
#### b) 不完美清单
+ 电池已识别并可判断电池充放电状态以及计算剩余时间，但电池百分比无法显示 1%
+ 电脑自带的指纹识别器无法使用(可用 MacID 代替，反正 PC 笔记本上的指纹识别也不很好用) 0.44%
+ iMessage、FaceTime 和其他白果专用服务、硬件(如无痛升级)无法使用，以及时不时 iCloud 弹窗恶心人 0.55%
+ 黑苹果哪有完美的 233333333 0.01%

## 2. 显卡及显示器
前面已经说了，这台 ASUS Pro451L 使用了 Intel HD Graphics 4400，而这个显卡并不是那么好驱动。不过当时我驱动时 RehabMan 大神已经写出了 FakePCIID，可以到 <a href="https://bitbucket.org/RehabMan/os-x-fake-pci-id/downloads">BitBucket</a> 下载，FakePCIID 还可以帮助驱动 Broadcom 网卡蓝牙等，这里不再细说。安装好 FakePCIID 后再 Clover 里面注入 ig-platform-id
我当时还顺便把 EDID 给注入了(ig-platform-id 和 EDID 的具体注入方法远景一堆堆)，
```plist
<key>CustomEDID</key>
<data><!-- BASE64 ENCODED DATA --></data>
<key>ig-platform-id</key>
<string>0x0a260006</string>
```
当时还没有显现什么，但就在期末考试前，我通过 <a href="https://bitbucket.org/RehabMan/os-x-intel-backlight/downloads">IntelBacklight.kext</a> 驱动了亮度调节。该驱动还内建了显示器。这样，当我插入 VGA 或 HDMI 外置显示器时，能够自动识别外置显示器。(HDMI 音频没有条件，未测试)

## 3. 电池检测的迂回方法
前面说过，电池无法正常显示百分比，但可以显示充电状态和剩余时间。所以不拿到电池百分比也无伤大雅，只要能随时知道剩余时间即可。我推荐大家使用 <a href="http://www.coconut-flavour.com/coconutbattery/">coconutBattery</a>，在它的设置中选中 Launch at startup 和 Show icons in menu bar，在中间的 Appearance in menu bar 文本框中输入`%r%s`，这样它就能在状态栏上显示电池剩余时间了。
<figure style="text-align: center; line-height: 1;">
<img style="text-align: center" src="https://blog.chickger.pw/content/uploadfile/201701/e4e51484896683.png" alt="剩余时间"></img><br>
<span style="font-size: 8pt; color: grey;">电池剩余时间显示</span>
</figure>
但是对于这台电脑，系统会自动认为不插电时就是低电量电池，可能是因为始终识别为 0% 电池百分比。可以通过删除报警提示程序来掩盖这个问题。
```bash
sudo rm -f /System/Library/CoreServices/Menu Extras/Battery.menu/Contents/Resources/lowBatteryWarning # 前提请关闭 SIP 的文件系统保护
```

## 4. 触摸板手势高仿
终于说到大头了，这是今天刚刚做到的。我们知道 Mac 之所以令人喜爱，有一个重要的地方就是它的触摸板。而 ASUS Pro451L 所使用的 ELAN 触摸板算是上天眷顾的幸运儿。这种触摸板驱动的开发文档和第三方 Linux 开发者历史经验比较多，所以很快被移植到了 Mac 上。主要使用的驱动为 ApplePS2SmartTouchPad.kext。安装了这个驱动后，多指手势已然实现。但是这里面的手势和白苹果的手势基本不相同。要知道搞黑苹果的用惯了 macOS 是迟早要入果坑的。不能把我妈的习惯给搞坏了。我们知道在白苹果上：
+ 三指或四指上滑打开 Mission Control
+ 三指或四指下滑打开应用程序 Exposè
+ 三指或四指左右滑动切换桌面
+ 五指一捏打开 Launchpad
+ 五指散开显示桌面
+ 双指从右边缘向左打开通知中心

这些也就是黑苹果上能够高仿的手势了，通知中心那个亲测容易导致误操作。而且当打开了 Mission Control 时需要下滑关闭，Launchpad 等等同理。所以我设置的是：
+ 三指上滑或下滑开关 Mission Control
+ 三指下滑开关应用程序 Exposè
+ 五指捏散开关 Launchpad
+ 五指点击显示桌面
+ ~~右边缘向左打开通知中心~~ (易误操作，除非经常访问通知中心否则请勿开启)

那么如何设置呢？这需要修改触摸板驱动的 Info.plist。建议使用 PlistEdit Pro 修改。

<p style="color: red;">!! 以下内容建议对照 ApplePS2SmartTouchPad.kext/Contents/Info.plist  的结构阅读！</p>

首先从树形结构的 Root 触发，找到 IOKitPersonalities $\xrightarrow{}$ Smart-Pad $\xrightarrow{}$ Preferences。这下方有多种手势。可以根据自己的需要修改，基本上只要会了英语单词都能看懂，在此先给一个文档的链接，在这个链接中有详细的介绍：<a href="http://forum.osxlatitude.com/index.php?/topic/5966-details-about-the-smart-touchpad-driver-features/">传送门</a>（英语）

在该驱动中，不同的手势对应的不同动作是通过数字编号来定义的，比如我设置的 5FingersPinchAction 的值为 3，这就定义了五指捏合或散开则开关 Launchpad。关于哪个键值对应哪个手势，我从外网链接回来大家看看。

<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-50400800-1461153149.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-01840600-1461159119.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-51977900-1461164551.png"></img><img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-30506100-1461164558.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-20150000-1461168882.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-19740900-1461168889.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-78425700-1461170261.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-26361600-1461171972.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-86224500-1461170630.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-17071100-1461171339.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-03574300-1461171370.png"></img>
<img src="http://forum.osxlatitude.com/uploads/monthly_04_2016/post-7370-0-13536800-1461171913.png"></img>

关于哪个数字对应什么，默认的从那个文档中就有，但是有些部分不好懂，加上部分读者的英语水平也像我一样渣，那我就献个丑，给大家翻译 + 注释一番。

+ 0: 禁用该手势
+ 1: 应用程序切换器 (相当于 ⌘⇥ Cmd+Tab)
+ 2: 退出当前程序 (相当于 ⌘Q Cmd+Q)
+ 3: 开关 Launchpad
+ 4: 开关 Mission Control
+ 5: 开关 Dashboard
+ 6: 向左一个桌面或全屏程序 (相当于 ⌃← Ctrl+←)
+ 7: 向右一个桌面或全屏程序 (相当于 ⌃→ Ctrl+→)
+ 8: 应用程序窗口 (相当于在 Dock 图标上右键显示所有窗口)
+ 9: 最小化当前程序
+ 10: 全屏当前程序
+ 11: (在浏览器中)后退
+ 12: (在浏览器中)前进
+ 13: 显示桌面
+ 14: 调出通知中心 (不好用)
+ 15: 显示简介 (相当于 ⌘I Cmd+I)
+ 16: 显示/隐藏 Dock
+ 17: 调出通知中心
+ 18: 重置缩放 (就是把缩放设置为 1)
+ 19: 打开 Finder
+ 20: 弹出强制退出对话框 (相当于 ⌥⌘⎋ Opt+Cmd+Esc)
+ 21: 设定该手势为鼠标右键
+ 22: 设定该手势为鼠标中键
+ 23 ~ 34: Mac 键盘扩展功能键 F13 ~ F24
+ 35: 设定该手势为左键

看了这个是不是有点晕乎？其实很简单，配合文档附带的图片，想修改哪里就修改哪里。我将我的修改版上传，你们可以随意使用。
下载链接: <a href="https://pan.baidu.com/s/1nu9wkml">传送门</a> 提取码: 9ihp

## 5. Ending
黑苹果本身并不难，主要考验的就是耐心和恒心，只要坚持下来，该换硬件的乖乖换硬件，不用换硬件的自己爬贴学习，攒机之前多方面考察，迟早会有一台 99% 以上完美的黑苹果的。