---
layout: post
title: 使用Markdown写博过程中存在的问题及解决方法
category: others
tags: others
keywords: Markdown
description: Markdown使用过程中存在的问题及解决方法
---

&emsp;&emsp;用了一段时间的Markdown，整体感觉挺趁手，但由于Markdown语法是通过标签来排版，使用过程中会存在一些排版的问题，下面总结一下使用Markdown半年来遇到的问题及解决方案（由于使用时间有限，还有很多问题没有遇到，或者有些问题有更好的解决方案，欢迎大家批评指正）：

- **文档中带Html标签：**我的博客是在Github上用模板搭建的，用到了jekyll静态语言，由于markdown是支持html标签的，所以在写博过程中经常会遇到带网页标签的文章内容和Markdown排版关键字冲突，比如我今天写的一篇博客"Android中<include>标签的使用及注意事项"，在自己的博客上直接显示错乱，最后不得不改改掉标题为“Android中include标签的使用及注意事项”，并将博文中的"<include>"全部改成"**include**"，做到和Markdown支持的标签有区别后才能够正常显示，在简书上正文虽然能够正常显示，但标题同样也会有这个问题：

![简书标题对网页标签的支持](http://ww2.sinaimg.cn/large/6d17e381gw1eotmnirysij20rc06o0tf.jpg)

- **缩进：**每段文章我都会习惯性地做首行缩进，但如果直接敲空格和TAB，虽然能够在写作的地方看到效果，但发布到博客上根本是没有缩进的效果的，这需要用到"&emsp"标签，通过在段首加上&emsp;&emsp"便能实现首行缩进的功能。

- **代码排版：**Markdown上的代码排版很简单，将代码粘贴进来，然后选中整段代码，按Tab键缩进，就可以完成代码排版，但有一个问题，如果将两段代码放在一块但又需要分开显示时，问题就来了，比如我需要下面这种效果：

![排版效果](http://ww1.sinaimg.cn/large/6d17e381gw1eotn8q56imj20m608nq3e.jpg)

&emsp;&emsp;如果在两段代码中不做分隔，上图的显示效果就不会有中间的白条分隔了，两段代码直接成一段了：

![这样排版达不到效果](http://ww1.sinaimg.cn/large/6d17e381gw1eotnaru2snj20ih05w74p.jpg)

&emsp;&emsp;我通常是通过在两段代码间加上markdown支持的标签来做分隔，比如"&emsp"，将正文写成下图这样就可以达到想要的效果了：

![这样排版才OK](http://ww4.sinaimg.cn/large/6d17e381gw1eotnc4xo6ej20hj074mxk.jpg)

- **图片：**markdown中链接图片（包括gif）很简单，直接使用![markdown中调用图片](http://ww4.sinaimg.cn/large/6d17e381gw1eotneer2syj204900o742.jpg)即可，但如何将图片生成链接是比较麻烦的，我通常是通过图床工具，将图片上传到图床，然后将生成的链接拷贝过来即可，墙裂推荐[微博是个好图床](https://chrome.google.com/webstore/detail/%E5%9B%B4%E8%84%96%E6%98%AF%E4%B8%AA%E5%A5%BD%E5%9B%BE%E5%BA%8A/pngmcllbdfgmhdgnnpfaciaolgbjplhe?utm_source=chrome-app-launcher-info-dialog)，在chrome上安装好这个插件后，直接将图片拖进去即可生成图片链接：

![微博是个好图床](http://ww4.sinaimg.cn/large/6d17e381gw1eotnj058igj20oe0qsab1.jpg)

&emsp;&emsp;**注意：**如果你是用RTX截图，最好不要保存为png格式的图片，这个在图床上是上传不成功的（很多网站都不支持它，具体为什么我也没研究），还有一些其它的图床也是可以的，只是需要注册，需要大空间时要收费，最主要是没有微博是个好图床方便。

- **gif:**Github上的开源项目，ReadMe.md是也支持Markdown语法的，通常会看到很多开源项目的ReadMe中有动态演示效果，看到这个项目的人一目了然，非常方便，gif本身也是一种图片格式，在Markdown中引用时和正常图片的引用一样，但需要专门的工具生成gif格式的图片才行，在这里墙裂推荐[LICEcap](http://www.appinn.com/licecap/)，它是一款windows上的录屏软件，录制后保存的格式为gif，体积小并且同样也可以将其拉到图床上生成链接。

- **文档通用问题：**如果你用Markdown写好文章，需要放在好几个博客上，但有的博客又不支持Markdown语法（比如CSDN），显然重新排版文章是不科学的，如果文章后续有改动，更新的时候岂不是得改好几个地方？可以将Markdown转换成html或者PDF文件来解决这个问题，具体转换方式可以在网上查找，如果你用的是MarkdownPad，直接Export就可以了。

- **段间排版：**我在写博客的时候，段与段之间需要空一行，否则在博客上的显示效果是博文根本没有按段区分，揉在一起了。但简书上就不会存在这个问题，我想这和每个平台对Markdown语法的解析有关，但建议在用Markdown写文章的时候，段与段之间用空行分隔。





