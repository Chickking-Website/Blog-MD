更好的 Markdown for emlog 解决方案
---
目录:

[TOC]

上一篇博文说到了让 emlog 支持 Markdown，但是在实际应用中我发现效果并不是非常理想，如在文章中输入`\r`会换行，以及不支持页面的 Markdown Parse。于是想去改进一下，读了 Editor.md 的文档，我决定基于 Editor.md 实现 Markdown 的 Parser。

## 1.安装并配置 Editor.md
安装 Editor.md 十分简单，我就不多说了，大家都会的。  
重点在于配置 Editor.md，如何配置呢？这取决于你需要怎样的功能，这里推荐你去看看它的 [Examples](https://pandao.github.io/editor.md/examples/index.html)。
我的需求是 HTML in Markdown、TOC、以及输出 HTML，流程图和时序图以后恐怕也会用到。我的配置如下:
```html
        <script type="text/javascript">
			var testEditor;

            $(function() {
                testEditor = editormd("test-editormd", {
                    width   : "100%",
                    height  : 640,
                    syncScrolling : "single",
                    htmlDecode : true,
                    sequenceDiagram : true,
                    flowChart : true,
                    saveHTMLToTextarea : true,
					tocm : true,
                    tocContainer : "",
                    tocDropdown   : false,
                    path    : "https://static.chickger.pw/js/editor.md/lib/"
                });
            });
        </script>
```
## 2.添加 Parser
首先，我们在 emlog 的`admin/views`中上一个博文提到的四个 php 文件中 form 里的合适位置处插入一个 `<textarea>`，并命名其 name 和 id 为 `content`。通过审查元素我们发现: Editor.md 的 Markdown Preview Div 的 ClassName 为 `markdown-body editormd-preview-container`，且整个 HTML 中拥有此 ClassName 的有且只有这一个。所以我们可以通过令这个 `<textarea>` 的 `innerText` 属性等于 Preview Div 的 `innerHTML` 属性来实现 Parse。于是代码如下:
```javascript
var applyer = document.getElementById('content');
var source = document.getElementsByClassName('markdown-body editormd-preview-container')[0];
applyer.innerText = source.innerHTML;
```
## 3.获取 Markdown 原文
由于 emlog 本身不支持 Markdown，所以我通过 github 转存 Markdown 文件，而 Parse 后无法获取源文件，不够方便，于是可以判断如果文章发表时间在第一篇 Markdown 文章之后则去 github 获取 Markdown 原文并进行 HTML 转义。代码如下:
```
<textarea class="editormd-markdown-textarea" style="display:none;"><?php $logDay = gmdate('Y-n-j', $date);$logSec = strtotime($logDay);if ($logSec > 1476439200) echo htmlspecialchars(file_get_contents("https://chickking-website.github.io/Blog-MD/" . 'post-' . $_GET['gid'] . '.md')); else echo $content; ?></textarea>
```
而且这样还实现了我修改文章无需复制修改后的原文去管理后台，只要 push 到 GitHub 后打开管理后台之后保存即可。  
这样就近乎完美实现了 emlog 支持 Markdown 这一目标。