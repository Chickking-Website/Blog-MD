<!-- 标题：【Hack】AppleALC 驱动 CX20751/2 声卡实战 -->
2017 了哈，新年新气象。趁着 1 月 1 日，一番捣鼓把一度以为无解的 Mom's 电脑上的声卡 CX20751/2 驱动了。感觉心里的一块大石头落地了呢，毕竟给她装了黑苹果却让她无法使用声卡怎么说也不是个事啊，愧疚感终于消除了！！
## 1. 准备工作
首先我们需要去下载一个 macOS 下的 App 叫做 <a href="https://github.com/Mirone/AppleHDAPatcher">AppleHDA Patcher.app</a>，点击蓝链可以去 GitHub 下载。

然后在老妈的电脑上运行 AppleHDA Patcher.app，我选择右侧 Laptop 栏中的  Conexant CX20752（CX20751/2 声卡和 CX20752 声卡使用同一芯片）。泥萌就选择自己的声卡吧。

<a href="https://blog.chickger.pw/content/uploadfile/201701/37d01483277857.png"><img src="https://blog.chickger.pw/content/uploadfile/201701/thum-37d01483277857.png"></img></a>

然后点击 patch AppleHDA，会在桌面上生成一个文件夹叫做 MironeAudio。我为了不让 Mom's 电脑多一些她不需要的东西就没在她的电脑上装 Xcode。所以下面的步骤都是在我自己的电脑上进行的。

先在终端里面找到需要的路径，输入终端代码。
```bash
$ git clone https://github.com/vit9696/AppleALC.git
```
## 2. 主要步骤
重头戏来了。

#### Step 1. 处理文件
这一步比较绕，注意不要删错了文件。

进入 AppleALC/Resources/ 目录，删除除了你自己声卡型号之外的所有声卡型号，比如我就删除除了 CX20751_2 以外的所有声卡型号。

还记得之前我们保存的 MironeAudio 文件夹吗？进入目录 MironeAudio/14f1510f**（此处随声卡型号的不同而不同）**/274.12**（此处随系统版本的不同而不同）**/Clover/aDummyHDA.kext/Content/Resources/，将 layout3.xml.zlib 和 Platforms.xml.zlib 拷贝到 AppleALC/Resources/CX20751_2**（此处随声卡不同而不同）**/ 下覆盖。

然后打开 Sublime Text 来编辑 Plist 文件。在 aDummyHDA.kext 的 Info.plist 中搜索 HDAConfigDefault，其下面有一个值叫做 Codec ID。例如我的：
```XML
<key>CodecID</key>
<integer>351359247</integer>
```
这个值要记下来，下一步用到。

在 AppleALC/Resources/CX20751_2/Info.plist 中，修改上面的 CodecID 为刚刚获取的数值。*(其实不修改理论上也可以，但有些机器不修改就无法驱动。)*

在多个 Comment 的键值中，找到 “Mirone - 你的声卡型号” 的一项，其下面的 ID 修改为 3，Path 修改为 Platforms.xml.zlib。
```xml
<key>Comment</key>
<string>Mirone - Conexant 20752</string>
<key>Id</key>
<integer>3</integer>
<key>Path</key>
<string>layout3.xml.zlib</string>
```
再到 MironeAudio/14f1510F/274.12/full Patched AppleHDA/AppleHDA.kext/Contents/PlugIns/AppleHDAHardwareConfigDriver.kext/Contents 下，打开 Info.plist。将 HDAConfigDefault 键值下面 array 的内容复制下来，别告诉我你不会判断 array 的开始和结束，读 XML 应该是黑苹果的基本功了。

```XML
<key>HDAConfigDefault</key>
<array>
	<dict>
		<key>AFGLowPowerState</key>
		<data>
			AwAAAA==
		</data>
		<key>Codec</key>
		<string>Mirone - Conexant CX20752</string>
		<key>CodecID</key>
		<integer>351359247</integer>
		<key>ConfigData</key>
		<data>
			AWccEAFnHUABZx4hAWcfAQF3HCABdx0AAXceFwF3H5ABhxwwAYcdkAGHHoEBhx8BAaccQAGnHQABpx6gAacfkA==
		</data>
		<key>FuncGroup</key>
		<integer>1</integer>
		<key>LayoutID</key>
		<integer>3</integer>
	</dict>
</array>
```
然后将其粘贴到 AppleALC/Resources/PinConfigs.kext/Info.Plist 中，使其替换原有的 HDAConfigDefault 键值下数组的内容**（会有多个，不要替换多也不要替换少，会选中大片内容）**。

这样第二步就结束了。

#### Step 2. Xcode 编译打包成 kext 文件
现在打开我们修改完了的 AppleALC 文件夹，打开 AppleALC.xcodeproj。建议你使用 Xcode 最新版本。请确保命令行工具安装完整。

**检查之前所有的工作准备无误后**（非常重要，不然后来驱动不了你都不知哪里出了错），在 Xcode 的工具栏上点击 Project $\rightarrow$ Archive。

<a href="https://blog.chickger.pw/content/uploadfile/201701/63841483281442.png"><img src="https://blog.chickger.pw/content/uploadfile/201701/thum-63841483281442.png"></img></a>

上面读条完毕后，会弹出一个窗口，你们之前没有 Archive 过所以会只有一项，点击右侧的 "Export..." 按钮，将其导出到你希望的保存的地方。然后进入 AppleALC 2017-01-01 22-40-06 文件夹（随时间的不同和个人重命名可能有差别），依次打开 Products/Library/Extensions/。可以看到 AppleALC.kext 现在将其复制到一个文件夹里面，暂且称之为 ExportedKext。

前往之前得到的 MironeAudio 文件夹，将 CodecCommander 目录下的 CodecCommander.kext 复制到 ExportedKext 文件夹里面。此 kext 可以防止睡眠唤醒后无声。

#### Step 3. 黑苹果机上配置 Clover

在黑苹果上 Mount EFI 分区，进入 Clover 的文件夹。现将刚刚复制出来的两个 kext 复制到 Clover/Kexts/10.x(根据自己系统版本)下和 Other 下。（据说 macOS Sierra 10.11.2 版本下 Clover 无法注入 Kext？那就安装到 SLE 或者 LE 下）

然后就需要修改 config.plist 了。强烈建议先做备份，很重要，万一改毁了还有个备用的。而且个人不推荐用 Clover Configurator 来修改 config.plist，因为我用那个修改完后，在好几台机子上引导崩了好几次。建议用编辑器手动修改。

首先在 Boot 键值下面找到 Arguments 键值（也就是修改 Boot Args），加一个
```css
-alcbeta
```

然后在 Devices 键值下的 Audio 键值下将 Layout ID = 3 注入。代码如下：
```xml
<key>Audio</key>
	<dict>
		<key>Inject</key>
		<integer>3</integer>
	</dict>
```
当然某些 DSDT 大神们也可以把这个注入 Layout ID 和防止唤醒无声的补丁都在 DSDT 里面解决了，效果会更好。不过我等蒟蒻还是算了，除了除错 DSDT 啥也不会~~我是不是废了……

#### Step 4. 重启看效果
明眼人一看这就是在讲(còu)玄(zì)学(shù)。不过重启一下看到音量图标一下子出来了，按下 Fn+F12 也能听见音量调节的嘟嘟声也是蛮令人激动的有木有。如果没有驱动上请赶紧检查上面的步骤有没有做错 or 面壁思过想想 2016 自己怎么搞的 RP-- :)

## 3. 评价
不得不说 vit9696 大神放出的 AppleALC 实在是声卡驱动界的一颗重磅炸弹。有了 AppleALC+AppleHDAPatcher 简直就是天下声卡任我驱的组合了。目前貌似声卡没什么是这个无法驱动的了。关键是这个不怕系统升级，只要 Clover 还能注入 kext 并关闭 SIP（其实不能注入也可以装到系统里面，不过如果不能关 SIP 就麻烦了……不过那样连系统也进不去），AppleHDA 基本就能永久驱动。

还有 @Menci，你的 <a href="https://moeditor.org/">Moeditor</a> 出奇的好用。
