
本文收集 System.Net.Sockets.SocketException 异常错误码为 10106 导致无法联网的问题

<!--more-->


<!-- CreateTime:2022/9/1 19:30:56 -->

<!-- 发布 -->

这里的 10106 是 Win32 的 Socket 错误码，可以从 [Windows Sockets Error Codes (Winsock2.h) - Win32 apps Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2 ) 文档了解到 10106 对应的是 WSAEPROVIDERFAILEDINIT 错误，描述如下

> Service provider failed to initialize.
>   The requested service provider could not be loaded or initialized. This error is returned if either a service provider's DLL could not be loaded (LoadLibrary failed) or the provider's WSPStartup or NSPStartup function failed.

这一层出现问题，和 .NET 的关系不大，其他技术也可能会遇到相同的问题。用户端遇到此问题，基本都是用户的电脑环境的网络配置不正确。修复方法基本是重置网络，如以下方法

解决方法：

命令行输入以下代码，然后重启机器即可

```
netsh winsock reset
```

相关参考博客：

- [System.Net.Sockets.SocketException: 无法加载或初始化请求的服务提供程序 - 二笔青年](https://blog.clso.fun/posts/2019-09-29/137.html )
- [socket 10106问题解决日记_忆常的博客-CSDN博客](https://blog.csdn.net/wjtxt/article/details/10500817 )
- [服务器上tcp程序突然连接不上报错 System.Net.Sockets.SocketException (0x80004005): 无法加载或初始化请求的服务提供程序。 - Ace001 - 博客园](https://www.cnblogs.com/xuejianxiyang/p/14512955.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。