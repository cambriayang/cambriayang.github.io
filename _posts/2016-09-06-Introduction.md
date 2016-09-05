# MWeb - 专业的 Markdown 写作、记笔记、静态博客生成软件

## 专业的 Markdown 写作支持

* 极简 UI、Dark Mode、漂亮的 Markdown 语法高亮、列表缩进优化，提供 5 种主题选择。
* 除了支持基本的 Markdown 语法外，还支持大量 Markdown 扩展语法：Table、TOC、MathJax、Fenced code block、任务列表（Task lists）、顺序图和流程图、Footnote 等。
* 支持 Typewriter Scrolling（打机滚动模式）`View` - `Typewriter Scrolling`。
* 支持发布和**更新**到：Wordrpess 博客、支持 Metaweblog API 的博客服务、Wordpress.com、Evernote 和印象笔记、Blogger、Tumblr。请在 `Preferences` - `Publishing` 增加发布服务，然后点击软件右上角的分享按钮即可看到所增加的发布服务。
* 支持即时预览并提供 6 种预览主题，其中二种和静态博客主题相对应，也就是说您在写博客时可以即时预览大概效果！所有主题效果都支持导出为 HTML、PDF。快捷键 `CMD + R` 或 `CMD + 4` 打开即时预览窗口。
* 编辑器和实时预览都支持大纲视图，长文档时跳转非常方便。

## 设计为两种模式

* 外部文档模式：用于新建、打开和编辑外部 Markdown 文档。也支持引入外部文件夹到 MWeb 中管理。
* 文档库模式：用分类树管理文档，可以把文档设为多个分类，用于记笔记和静态网站生成。

`CMD + E` 或使用菜单 `View` - `Open External` 可打开外部文档模式。

`CMD + L` 或使用菜单 `View` - `Open Library` 可打开文档库。

文档库模式和外部模式都支持**全文搜寻（Full Text Search）**，都可以用拖放或粘贴插入图片并直接显示。`CMD + V` 粘贴为JPG格式，`CMD + Shift + V` 粘贴为PNG透明格式。

外部模式引入 Octpress、Jekyll 等静态博客的文件夹后也支持拖放或粘贴插入图片和实时预览，详细请参考：[引入文件夹到 MWeb 中管理，支持 Octpress、Jekyll 等静态博客拖拽插入图片和实时预览](http://zh.mweb.im/mweb-1.4-add-floder-octpress-support.html title)

-----
<table>
    <tr>
        <td>Foo</td>
        <td>HH</ta>
    </tr>
    <tr>
	     <td>MM</td>
    </td>
    &copy;2016
    &lt; &amp;&gt;  
 </table>
	> **Test**
	 This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
		>> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
	> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
*Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.<br />
*Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.<br />
*一列表项包含一个列表区块：

	<let myName: String = "Cambria">
	Here is an example of AppleScript:

    tell application "Foo"
        beep
    end tell
<div class>
	&copy; 2016 Foo Corporation
</div>
------


## 文档库模式用于记笔记

文档库模式使用分类树组织和管理文档，支持拖放或粘贴插入图片并直接显示，插入非图片则会生成连结。
支持把 Markdown 或文本文档导入到文档库，也支持把整个分类或者文档（可选多个）导出为 HTML、PDF、Markdown。

更详细的信息请看：[MWeb 文档库模式详细说明](http://zh.mweb.im/mweb-document-library.html "MyTitle") Yes, wait.

*Visit* [Daring Fireball] for more information.
[Daring Fireball]: http://daringfireball.net/

**I get 10 times** more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].
  
  [1]: http://google.com/        "Google"
  [2]: http://search.yahoo.com/  "Yahoo Search"
  [3]: http://search.msn.com/    "MSN Search"
  ![屏幕快照 2016-09-04 21.40.12](media/14269480090478/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-04%2021.40.12.png "optional")

``There is a literal backtick (`) here.``
`There is a literal backtick (fas) here.`

<http://baidu.com/>
&#64;



## 文档库模式用于静态博客生成

一键把分类生成静态博客，目前可选二个主题，支持自定主题。只要填入 Disqus、多说提供的代码即可以为博客增加评论功能。可勾选让网站支持 MathJax 和顺序图、流程图。

更详细的信息请看：[MWeb 生成静态博客详细说明](http://zh.mweb.im/mweb-static-blog-generator.html)、[静态博客功能增强](http://zh.mweb.im/mweb-1.4-static-blog-extension.html)



