
本文记录在 2403 麒麟系统上，使用 wmf2gd 转换 wmf 图片时提示 ERROR: font.c (1334): wmf_ipa_font_map: failed to load any font! 错误问题

<!--more-->


<!-- CreateTime:2025/07/25 07:13:48 -->

<!-- 发布 -->
<!-- 博客 -->

以下是我的转换命令

```
./wmf2gd -t png -o 1.png image.wmf
```

报错信息如下

```
ERROR: font.c (1334): wmf_ipa_font_map: failed to load *any* font!
```

查阅本文下方的参考文档了解到，这是因为 wmf 尝试去读取 gsfonts 字体失败

我尝试使用 `ls /usr/share/fonts/type1` 命令了解到我当前的麒麟系统没有安装 gsfonts 字体

于是我就直接去 ubuntu 源下载 gsfonts 字体，下载代码如下

```
wget https://cn.archive.ubuntu.com/ubuntu/pool/universe/g/gsfonts/gsfonts_8.11%2Burwcyr1.0.7~pre44-4.5_all.deb
```

下载之后，使用如下代码进行安装

```
sudo dpkg -i gsfonts_8.11+urwcyr1.0.7~pre44-4.5_all.deb
```

安装之后即可正常使用 wmf2gd 转换 wmf 为 png 图片

这个过程中，我尝试用了 `--wmf-fontdir` 参数，发现是没有什么作用的，其命令如下

```
./wmf2gd -t png -o 1.png --wmf-fontdir=/usr/share/fonts/type1/gsfonts image.wmf
```

但如果自己将 gsfonts 解压出来，将 pfb 等字体放在自己的文件夹，再使用 `--wmf-fontdir` 设置到自己的文件夹，这是有用的

本文的所有方法对 wmf2svg 等工具同样生效

参考文档：

[ERROR: font.c (1334): wmf_ipa_font_map: failed to load *any* font! · Issue #25 · kakwa/libvisio2svg](https://github.com/kakwa/libvisio2svg/issues/25 )

[Convert WMF file fails - Legacy ImageMagick Discussions Archive](http://www.imagemagick.org/discourse-server/viewtopic.php?t=18987 )

[Bug #629153 “wmf2XXX tools fail due to defoma font issue” : Bugs : libwmf package : Ubuntu](https://bugs.launchpad.net/ubuntu/+source/libwmf/+bug/629153 )

[[solved] wmf2svg: failed to load *any* font - debianforum.de](https://debianforum.de/forum/viewtopic.php?t=122797 )

[gsfonts usage in ImageMagick - Legacy ImageMagick Discussions Archive](https://legacy.imagemagick.org/discourse-server/viewtopic.php?t=35933 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。