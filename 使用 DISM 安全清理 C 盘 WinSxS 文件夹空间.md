
本文将介绍如何使用系统内置 DISM 工具进行安全清理 C 盘空间，清理 WinSxS 文件夹里面的可回收删除的程序包空间

<!--more-->


<!-- CreateTime:2023/5/6 14:19:19 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，先使用管理员权限打开 CMD 或 PowerShell 命令行窗口。在 Win11 下，可右击开始菜单，点击终端管理员打开管理员权限的 PowerShell 命令行窗口

先查看 WinSxS 文件夹里面的可回收删除的程序包数量和上次清理的时间，以及 WinSxS 文件夹所占用的实际空间，从而决定是否进行下一步动作

在命令行窗口输入以下命令用来[确定 WinSxS 文件夹的实际大小](https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/determine-the-actual-size-of-the-winsxs-folder?view=windows-11)

```
Dism.exe /Online /Cleanup-Image /AnalyzeComponentStore
```

执行之后输出的内容大概如下：

```
部署映像服务和管理工具
版本: 10.0.22621.1

映像版本: 10.0.22621.1555

[==========================100.0%==========================]

组件存储(WinSxS)信息:

Windows 资源管理器报告的组件存储大小 : 9.09 GB

组件存储的实际大小 : 6.21 GB

    已与 Windows 共享 : 5.19 GB
    备份和已禁用的功能 : 1.02 GB
    缓存和临时数据 :  0 bytes

上次清理的日期 : 2023-05-03 15:10:13

可回收的程序包数 : 1
推荐使用组件存储清理 : 是
```

各个项的说明如下：

- Windows 资源管理器报告的组件存储大小：Windows 资源管理器计算的 WinSxS 文件夹的大小值。 此值不会考虑 WinSxS 文件夹中使用的硬链接。
- 组件存储的实际大小：此值考虑 WinSxS 文件夹中的硬链接。 它不会排除通过硬链接与 Windows 共享的文件。
- 与 Windows 共享：此值提供使用硬链接的文件的大小，这些文件使用硬链接以便同时显示在组件存储和其他位置（为便于正常操作 Windows）中。 这包含在实际大小中，但不应视为组件存储开销的一部分。
- 备份和禁用的功能：这是为响应更新组件中的故障或提供启用更多功能的选项而保留的组件的大小。 它还包括组件存储元数据和并行组件的大小。这包含在实际大小中，是组件存储开销的一部分。
- 缓存和临时数据：这是组件存储在内部用于加快组件服务操作速度的文件的大小。 这包含在实际大小中，是组件存储开销的一部分。
- 上次清理日期：这是最近完成组件存储清理的日期。
- 可回收包数：这是组件清理可以删除的系统中被取代包的数量。
- 建议的组件存储清理：这是组件存储清理建议。 如果执行清理过程可以减少组件存储开销的大小，则建议进行清理。

如看到推荐清理存储，或者是自己感觉想清理看，则可以执行以下命令进行清理。清理是安全的，清理稍微缓慢，多跑跑也只是最多浪费点时间而已，不会损坏系统

```
Dism.exe /Online /Cleanup-Image /StartComponentCleanup
```

执行以上命令即可进行安全的清理。清理完成可以再次执行 `Dism.exe /Online /Cleanup-Image /AnalyzeComponentStore` 看看实际清理多少空间

如果觉得以上的命令清理的不够多，还可以再加上稍微危险的 `/ResetBase` 命令，此命令在 Windows 10 或更高版本的运行版本中，将删除组件存储区中每个组件的所有替换版本。换句话说就是丢失卸载补丁的功能，使用 /ResetBase 选项运行 /StartComponentCleanup 后，将无法卸载已安装的 Windows 更新。只有在确定自己当前的系统足够稳定时，也就是不需要卸载补丁才能使用如下命令

```
Dism.exe /online /Cleanup-Image /StartComponentCleanup /ResetBase
```

更多命令参数，请参阅 [DISM 操作系统包（.cab 或 .msu）服务命令行选项 Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/dism-operating-system-package-servicing-command-line-options?view=windows-11 ) 文档


参考文档：

[清理 WinSxS 文件夹 Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/clean-up-the-winsxs-folder?view=windows-11 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。