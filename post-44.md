【Mixed】Emlog 变量剖析+Keypad 开发心得+acme.sh 
---
目录:

[TOC]

这个博文写之前完全不知道怎么起标题……最近的技术学习比较杂乱，在这里略作笔记。
## 1. EMLOG 常用变量
最近萨德事件升温，之前对韩国没啥好感也恶感不多，这次实在是怒了，于是 Livere 直接成为祭品。之前一直没有换评论系统的原因是多说要根据每个网页修改 data-thread-key、data-title 和 data-url，之前还没找好解决方案。后来灵机一动，EMLOG 不是基于 PHP 的吗？扒一下 EMLOG 的变量不就 OK 了？于是前往了 EMLOG 的[官方 wiki](http://wiki.emlog.net/doku.php?id=tpldev)，发现了不少变量，但是似乎没同步更新，已经 out of date 了。扒了下程序代码，整理出了常用变量。如下表:  
![Emlog 变量表](https://static.chickger.pw/201703/Table_Of_EMLOGs_Variables.png)
这样，在多说的三个区域均输出相关三个变量的值即可。  
````html
<div class="ds-thread" data-thread-key="<?php echo $logid ?>" data-title="<?php echo $log_title; ?>" data-url="<?php echo 'https://'.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI']; ?>"></div>
````
## 2. Hello Electron, from Keypad!
项目的确是可以提升人的。我的第一个 electron 项目——[Keypad](https://github.com/Chickking/Keypad) 参上。目前还属于 Pre-beta 阶段，截止到3月12日最新版本为 0.9.1。Keypad 是基于 GPLv3 协议发布的，你可以在协议允许的范围内随意使用，目前没有 release 包。  
经过这个项目，我练习了表格布局，也真正接触到了日常前端开发需要的东西。这次我最大的收获是对 HTML DOM 的练习。也见识到了一些很棒的项目如 QRCode.js 等等。  
简单来说，这次的收获有:
- 表格布局的练习
- 阴影层、浮动 div
- CSS 的大量练习
- electron API 的学习
- HTML DOM 的使用
- JavaScript 的练习

我就挑主要的来介绍一下:  
### a) HTML DOM
这次编写 Keypad 对我来说受益匪浅，HTML DOM 的学习无疑是我最大的收获。下面主要讲讲 Keypad 中 HTML DOM 的应用。
<p style="color:red">推荐大家对照代码阅读。</p>

#### I. Substring
Keypad 的数字输入比较简单，而删除可以使用截取字符串的方式。Talk is cheap, show code:
````javascript
strNum.substring(0,strNum.length - 1);
````

#### II. window.location.hash
对于静态网页，而不是 php、asp 等动态网页等等，想获取输入就只能靠 URL 中 # 号后面的内容。而 DOM 恰好提供了函数来获取，那就是`window.location.hash`。  
因为这个代码中制作二维码完成后需要重置掉相关 div，我选择了使用重载网页的方式来重置，但是这样上方文本框中的号码也会不见。而通过 window.location.hash 即可传递电话号码。
index.html 中需要重载时的代码:
````html
<input class="closeBtn" type="submit" name="X" value="X" onclick="window.location.hash = document.getElementById('NumberShow').value;window.location.reload();"></input>
````
main.js 中在重载后获取号码的代码:
````javascript
//index.html 中将 <body> 的 onload 属性设置为 "getNumber()"
function getNumber() {
  var numSharp = window.location.hash;
  if (numSharp != null) {
      numSharp = numSharp.substring(1,numSharp.length);
      var field = document.getElementById('NumberShow');
      field.value = numSharp;
  }
}
````
<p style="font-weight: bold; color: red;">注意: window.location.hash 获取的数据是带 # 的，需要用 substring 截取字符串或通过正则匹配来获取参数。</p>
#### III. JavaScript 条件判断句
Keypad 中使用的 JavaScript 条件判断句型有 if......else 和 switch case。if......else 的用法比较简单:
````javascript
if (expression1 > < >= <= != == === expression2) {
	exec_if;
}
else {
	exec_else;
}
````
也可组合 if 和 else 组成 elseif 语句，即在 if 的判断为否时再判断。  
switch case 的用法也很简单:
````javascript
switch (i) {
	case 1:
    	exec_1;
    	break;
    case 'apple':
    	exec_2;
        break;
    default:
    	exec_other;
        break;
}
````
相信我无须解释了，这个是不是太过简单了些。  
#### IV. CSS 混合讲解
<p style="color: orange;">高能预警: 下面代码很多。</p>
首先，如果我们创建了一个悬浮窗，一般我们需要一个阴影背景，可以创建一个 div，在它的 CSS 里面加入如下代码:
````css
  /* 为了 JavaScript 的动态控制，我将 display: none; 写在了 div 的 style 属性里面。 */
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1040;
  background-color: #000;
  opacity: 0.7;
````
在 electron 中，为了让我们的 app 更像原生 app，可以加入`-webkit-user-select: none;`使得文字无法被选中。  
为了让我的 app 和 Mac 更搭调，我设置了背景半透明，在 app.js 里面设置好 `mainWindow = new BrowserWindow({transparent: true, frame: true})`后，加入一个 div，设置 class 为 background，根据网上的实现方法，在 css 文件夹下放入一个 SVG 文件叫做 blur.svg。加入以下代码:
````xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1"
     xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink"
     xmlns:ev="http://www.w3.org/2001/xml-events"
     baseProfile="full">
    <defs>
        <filter id="blur">
            <feGaussianBlur stdDeviation="14" />
        </filter>
    </defs>
</svg>
````
在 main.css 中加入了以下代码来实现背景图层:
````css
.background {
  background-color: #ededed;
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: -1;
  opacity: 0.9;
  filter: url(css/blur.svg#blur);
}
````
效果如下:
![半透明效果](https://static.chickger.pw/201703/Keypad_blur.png)

那个，electron 怎么打包来着？谁教教我啊，prebuilt  装不了，packager 又需要 prebuilt……

## 3. acme.sh
之前用 [sslforfree.com](https://sslforfree.com) 实现了获取证书，但我不满意，通过 acme.sh+cron 可以实现定时更新证书。
首先我在 DNSPod 获取了 token，写了个脚本。贴上来了:
````bash
#!/bin/bash
export DP_Id="1234"
export DP_Key="KEY0000"
~/acme.sh/acme.sh --renew --force --dns dns_dp -d example.com -d a.example.com
````
通过 cron，可以令他 60 天 renew 一次。
````cron
0 0 1 */2 * "~/getssl.sh" > /dev/null
````
先闪了，回头在写一些有的没的。
