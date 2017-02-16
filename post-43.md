<!-- 标题: 网站架构改造 -->

昨天是貌似是情人节？咳咳，别想那些没用的、与自己无关的节日，昨天是 ENIAC 诞生 71 周年，这才是正道。  
那顺手就给网站做个改造，这次也把之前的那几个改动整理整理，主要分为几个部分。

#### Part 1. GitHub 助力, 静态资源存放
我们知道，网站一般要将静态资源存放在单独的服务器上，由于资金原因*(其实就是穷)*，单独服务器是用不起，纯粹的静态资源比如图片什么的是不怕别人知道目录树的。于是干脆就想到了 GitHub，毕竟 GitHub 的服务可是杠杠的。  
由于当时想尝试一个评论系统，当时注册了一个新的 GitHub 帐户，创建新的仓库。看起来直接用 GitHub Pages 比较好，但是这样绑定域名后就无法使用 https 功能了，这可比较坑爹了。还好 nginx 可以做反向代理，这样就可以把请求先 https 后转发给 GitHub 了。  
但是这样有性能瓶颈，我的服务器很渣，所以这样静态资源请求速度很慢。所以不得以在保持了原来反向代理的基础上。设置了请求具体的文件名则直接 rewrite 到 raw.githubusercontent.com 去。  
先直接上代码:
```config
server {
    listen 443;
    server_name static.chickger.pw;
    index index.html;
    location / {
        proxy_pass https://raw.githubusercontent.com/Chickking-Website/Static-Assets/master/;
                if (!-e $request_filename)
                {
                        rewrite ^(.+)$ https://raw.githubusercontent.com/Chickking-Website/Static-Assets/master/$1 last;
                }
    }
}

server {
        listen 80;
        rewrite ^(.*)$ https://$host$1 permanent;
}
```
删节掉了 SSL 证书部分，你懂得。  
静态资源可以直接上传到 GitHub。  
<img src="https://static.chickger.pw/201701/GitHub-asset.png"></img>
#### Part 2. Let's Encrypt, 证书替换
原来用的沃通感觉药丸，Apple、Mozilla、Google 都不信任它和它的上级 CA —— StartCom 了。所以选择更换到 Let's Encrypt。这里安利一个网站 [sslforfree.com](https://sslforfree.com/)，通过 acme.sh 来获取 SSL 证书，我使用的是 TXT DNS 验证，这种办法还算好，不怎么麻烦，但是设置好 DNS 的 TXT 记录后要略等几分钟来验证。以及千万要用 Chrome 打开这个网站，Safari 貌似不支持 blob: 协议，结果无法下载证书……遇到这种情况直接保存页面源码在 Chrome 里打开即可。
#### Part 3. Web, 网站 UI 小修
UI 部分没有动太多，填自己之前选择用这个模板的坑就不多说了。首先添加了一个提示条，这个是从 [Jerry Qu](https://imququ.com/) 大神那里看到的，就是对时间较久之前的文章给予提示。用 PHP 可以轻松造这个轮子。  
<img src="https://static.chickger.pw/201701/Time_tips.png"></img>
就是这个东西。

以及还有查看本文 Markdown 版本这个大家也看到了，也是从 Jerry Qu 大神那里看到的。我会在今后展示更好的实现方法。
#### Part 4. Livere, 社会化评论
之前不得不再申请一个 GitHub 帐户的原因就是当时想试试 [Comm(ent|it)](https://commentit.io/) 这个脑洞大开的产品。它把用户评论保存在 GitHub 仓库里面。结果折腾了半天就是显示/保存不了评论，汗……有大神弄懂了欢迎来教教我这个蒟蒻。  
不过评论系统还是要有的。Disqus 在国内无法使用，Jerry Qu 的 Blog 里面做到了不翻墙获取/发出评论的功能，但是这位大神把代码和他博客的 NodeJS、CSS 等等全混在一起了，所以没有开源的轮子又不会自己造……  
多说也不好用，最后发现 [Livere](https://livere.com) 这个来自韩国的评论系统还不错，像 GitHub 登录等等那都是小 case，功能也十分齐全。  
不过还是必须说 Livere 的一大缺点是难以导出评论，到时候实在需要迁移应该可以把所有页面下的评论抓取下来做成和 Disqus 等兼容的格式，到底像轮子哥这样的“专业造轮子”型的人天下任我游啊。