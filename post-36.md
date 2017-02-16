# 1. 序言
相信大家已经都看到 10 月 5 日的 Steve Jobs 纪念网页了。Steve 是一位天才，他逝世 5 周年，我们没有理由不去纪念他。

于是我就已经把纪念网页放上去了，为什么直到 10 天后才写关于网页的内容呢？主要是当时比较忙。那么我就梳理梳理在网页中使用的东西。

友情提示，最好先打开[纪念页](http://www.chickger.pw/HomePage/2016/Steve%20Jobs/)，对照代码来阅读这篇博文，以取得最佳效果。
# 2. 全站变灰
纪念伟人嘛，总是要有点哀悼的气氛。全站变灰是少不了的了。针对全站变灰，我们可以设置一个单独的 CSS 文件，然后在 index.html 和博客模板的 header.php 里面都引入这个文件，我是将这个文件放在了 www 站点下面，然后在 blog 站下面建立软链接。这样就可以通过一个 / 来全部搞定了。
Code:
```html
<link rel="stylesheet" type="text/css" href="/special.css" />
```
在 CSS 里面，加入这一行代码，即可全站变灰。Code:
```CSS
html{-webkit-filter: greyscale(1)}
```
在平时也不用删除代码，只需将灰度值1改为0即可。

# 3. index.html 引导页
此页面与平时的引导页面并无太多不同，仅仅加入了全站变灰。

# 4. 纪念页面 rememberingsteve.html
重头戏来了，主要从这里说。
## i. 网页主结构
网页使用了 fullPage.js，但是由于在手机上容易出现撑破 div 和 Mac 下触控板滚动多次触发的问题，我禁用了滚动翻页。

网页主要的 CSS 和 JavaScript 从外部引入，保证了网页本身的大小较小。

## ii. 第一屏
打开网页后，映入眼帘的就是乔布斯的照片，而这张照片是作为 section1(id: firstpage) 的背景存在的，并且自适应，是通过 CSS 来实现的。Code:
```CSS
@media (min-width: 540px) {
    #firstpage{ background-image: url(SteveJobs1080p.jpg); background-repeat: no-repeat; background-size: 100%}
}
@media (max-width: 540px) {
    #firstpage{ background-image: url(SteveJobsMob.jpg); background-repeat: no-repeat; background-size: 100%}
}
```
当可视区域小于 540px 时，CSS 会自动加载移动版本的图片。

在页面中我加入了背景音乐。但是众所周知，在移动设备上不会自动加载背景音乐，而这不是我希望的。我在网络上寻找了许久，发现 github 上有一个 gist 可以达到 fake-autoplay 的效果，[传送门](https://gist.github.com/ufologist/50b4f2768126089c3e11)。

## iii.第 2、4、5、6、7、8 屏
这几屏中使用的技术大同小异。主要是一些关于乔布斯的资料等等，如名人的悼念等。而这些我选择使用 iframe 来显示。

为了实现 iframe 自适应高度，偶在 &lt;iframe&gt; 标签的 onload 中加入了 js 代码来实现 iframe 的高度自适应。以第五屏为例。Code:
```html
<iframe name="famouspeople" id="famouspeople" onload="this.height=famouspeople.document.body.scrollHeight" width=80% src="data:text/html;base64,PCFET...tbD4K"></iframe>
```

本来是将 iframe 中的 html 文件编码成 Data URI 来实现目录的精简化，但是在本地测试 OK 之后上传到服务器测试发现 iframe 不能自适应高度了。原来因为 Data URI 没有域，js 代码为安全起见不去执行，所以我不得不换回之前的办法：引用外部 html。

要说这几个 HTML 是如何制作的，除了第六屏都是 Base64 编码的图片是我一行行代码写的以外。其他都是用 macOS 自带的 TextEdit.app 导出 html 之后修改的。修改时主要加入 &lt;marquee&gt; 标签来实现滚动、在 CSS 中加入多个备选字体等。

字体方面我的首选字体是 Myriad Pro，但是由于收费等原因，网页不能自带该字体。也算是个小小的遗憾吧。不过如果用户安装了 Myriad Pro 字体那显示效果得点 32 个赞啊。

在 Steve 功勋墙页面里面，将乔布斯的产品横向滚动，而这些都是图片，我在图片的底部加入了发布时间，所以图片必须底部对齐。还是万能的 CSS 可以解决。Code:
```css
body{vertical-align: bottom;}
```

## iv. 第三屏
第三屏是一个视频，是时代先驱——乔布斯纪录片。而为什么我单独把这个写出来是因为这里有一个问题困扰了我许久。

纪念网页自带 BGM，而这个视频也有声音。这里就发生了冲突，即视频播放时音乐不会暂停，就会混音，极大地影响了用户体验。

我本来是想通过 &lt;video&gt; 标签的 onplay 和 onpause 属性中利用 js 来控制音乐的播放、暂停。但是很奇怪，完全无法控制。到 10 月 2 日的午饭时间，我一边吃着饭一边想这回事，突然想到在 fake-autoplay 的那个 gist 里面有涉及到 addEventListener，我一下子灵光乍现。立刻跑到书房里，直接掀开睡的正香的电脑的盖子，立刻在 main.js 里加入了这一段代码。Code:
```javascript
//音频的 id 为 bgmusic，视频的 id 为 hdvideo.

document.getElementById('hdvideo').addEventListener("play", function()
{
  document.getElementById('bgmusic').pause();
});
document.getElementById('hdvideo').addEventListener("pause", function()
{
  document.getElementById('bgmusic').play();
});
```
打开网页一测试，发现果然可以了，这样整个网页就基本做完了。剩下的事情就是打包上传到服务器。

# 5. 后记
事实证明，为自己的偶像做事情的确是效率极高的（手动滑稽），这也算是我这么懒的人少有的如此疯狂地做一件事。不过越做这些就越会发现自己的不足，从而得到提升。所以想锻炼自己写代码的能力就自己找个项目做做吧。一而能提升代码水平，二来也能增强自信。许多伟大的项目都是在不经意间诞生的。

这篇是我第一篇用 Markdown 写成的博文。使用的编辑器为 Menci 写的 [Moeditor](https://moeditor.org)，如果有兴趣可以去看看。

[更新于 18:55] 我去，Moeditor 输出的 HTML 的 CSS 会覆盖部分博客原有的 CSS。吓得我赶紧把源码 clone 下来改了一份……