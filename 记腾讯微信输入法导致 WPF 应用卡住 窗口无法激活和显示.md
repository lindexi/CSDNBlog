
本文记录我远程一位老师的设备，这位老师使用的是 Win7 Sp1 系统，现象是 WPF 应用的窗口无法激活，在 user32 的 SetForegroundWindow 或 NtUserShowWindow 方法卡住

<!--more-->


<!-- CreateTime:2024/11/21 07:10:00 -->

<!-- 发布 -->
<!-- 博客 -->

我收集到了多个进程的卡住的 dump 文件，分析到卡住有两个不同的堆栈

堆栈1：

```
 	user32.dll!_NtUserShowWindow@8()	未知
 	[托管到本机的转换]	
>	PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox = false) 行 3293	C#
 	PresentationFramework.dll!System.Windows.Window.Hide() 行 1358	C#
    业务代码
```

堆栈2：

```
>	user32.dll!_NtUserCallHwndLock@8()	未知
 	user32.dll!_SetForegroundWindow@4()	未知
 	[托管到本机的转换]	
 	PresentationFramework.dll!System.Windows.Window.Activate() 行 1467	C#
 	业务代码
```

从进程模块上可以看到有微信输入法的以下模块注入：

- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\CrashRpt1500.dll 1.05.0.0
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\dbghelp.dll 10.0.10150.0(debuggers(- dbg).150616-1659)
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\wetype_tip_core.dll 1.00.4.289
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\WeUIResource.dll 1.00.0.0

复现步骤：

反复进入可能会激活显示窗口和隐藏窗口的逻辑即可

解决方法：

卸载腾讯微信输入法

卸载完成之后，重启进程即可修复

相关问题： [记微信截图导致 WPF 应用卡住 窗口无法激活问题](https://blog.lindexi.com/post/%E8%AE%B0%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE%E5%AF%BC%E8%87%B4-WPF-%E5%BA%94%E7%94%A8%E5%8D%A1%E4%BD%8F-%E7%AA%97%E5%8F%A3%E6%97%A0%E6%B3%95%E6%BF%80%E6%B4%BB%E9%97%AE%E9%A2%98.html )

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。