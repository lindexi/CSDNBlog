
本文将告诉大家如何解决在 VisualStudio 2022 的 调试-窗口 里面找不到内存、 反汇编、 寄存器这三个调试工具的问题

<!--more-->


<!-- CreateTime:2024/1/11 20:14:56 -->

<!-- 发布 -->
<!-- 博客 -->

找不到的原因是没有启用地址级调试

只需要在“工具”（或“调试”）>“选项”>“调试”中选择“启用地址级调试” 然后进行调试即可看到

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具1.png) -->
![](http://cdn.lindexi.site/lindexi%2F2024111201747659.jpg)

开启之后，即可在 调试-窗口 里面找到内存、 反汇编、 寄存器这三个调试工具

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具0.png) -->

![](http://cdn.lindexi.site/lindexi%2F20241112017237286.jpg)

参考文档：[在调试器中查看寄存器值 - Visual Studio (Windows) Microsoft Learn](https://learn.microsoft.com/zh-cn/visualstudio/debugger/how-to-use-the-registers-window?view=vs-2022 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。