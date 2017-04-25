Let's enable HTTP/2
===
目录：

[TOC]
## HTTP/2 是什么？
HTTP/2，是超文本传输协议(Hypertext Transfer Protocol)的后续版本，上一版本是 HTTP/1.1。
## 为什么要开启 HTTP/2？HTTP/2 有什么优势
作为 HTTP/1.1 的下一个版本，HTTP/2 相比前代大幅度提高了 web 性能，且看 Akamai 提供的 [demo](https://http2.akamai.com/demo)。  
HTTP/2 大大提升了并发性能，并且在低延时的情况下具备着极为优越的响应速度。况且，开启 HTTP/2 也是网站实力和具有对新技术的追求的象征。以往许多大公司都将静态资源部署在单独的域名上，但开启了 HTTP/2 之后将不再需要这样做。这对于我这样的没钱买域名的站长来说是一大福音。
## 开启 HTTP/2 的要求
* nginx 服务器程序版本在 v1.9.5 以上，并且具有编译参数 `--with-http_v2_module`，通过 yum 安装的 nginx 具备此参数。
* OpenSSL 最好升级到最新版，我开启 HTTP/2 时版本是 1.0.1
* 网站需部署全站 HTTPS
* 面向人群使用现代浏览器
* 如果你和我一样使用 Cloudflare 的 CDN，可以先在 Cloudflare 打开它提供的 HTTP/2，然后再在服务器端配置。想装 X 或者服务器不方便开启 HTTP/2 的同学也可以只开 CDN 的 HTTP/2，只不过这样最后请求时还会走 HTTP/1.1，性能有所下降。

## 如何打开 HTTP/2？
首先对于没有编译参数的同学需要重新编译 nginx，像我这种懒人就直接 yum 安装最新的 nginx 了。  
当然对于喜欢折腾的同学这里也推荐