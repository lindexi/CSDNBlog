
本文将记录一个在 WPF 应用程序启动过程中的性能优化点。如果一个窗口需要设置 WindowStyle 属性，那么在窗口 EnsureHandle 之前，设置 WindowStyle 属性将会比在 EnsureHandle 之后设置快不少

<!--more-->


<!-- CreateTime:2023/8/19 16:13:03 -->
<!-- 标题： WPF 优化 EnsureHandle 启动性能 -->
<!-- 发布 -->
<!-- 博客 -->

在 dotTrace 里的一次测量在窗口 EnsureHandle 之后设置 WindowStyle 属性的时间大概是 200 毫秒，这个时间在启动过程中可以被认为是非常长的时间，而且这还是一个在 UI 线程上的时间。以下是 dotTrace 测量结果

<!-- ![](image/WPF 启动性能优化 在 EnsureHandle 之前设置 WindowStyle 提升性能/WPF 启动性能优化 在 EnsureHandle 之前设置 WindowStyle 提升性能0.png) -->

![](http://cdn.lindexi.site/lindexi%2F20238181613353535.jpg)

可以看到在窗口 EnsureHandle 之后设置 WindowStyle 属性，需要等待 Win32 窗口的响应。再阅读 WPF 源代码，可以看到在窗口 EnsureHandle 之后设置 WindowStyle 属性，就需要等待 HwndStyleManager 的 Dispose 方法。以下是 WPF 的源代码

<!-- ![](image/WPF 启动性能优化 在 EnsureHandle 之前设置 WindowStyle 提升性能/WPF 启动性能优化 在 EnsureHandle 之前设置 WindowStyle 提升性能1.png) -->

![](http://cdn.lindexi.site/lindexi%2F2023818161476079.jpg)

看起来这是非常合理的耗时，在 Win32 窗口创建出来之后，也就是对应 EnsureHandle 拿到窗口句柄之后，这时如果设置 WindowStyle 属性，就需要同步给到 Win32 窗口。为了同步设置给到 Win32 窗口，自然就需要等待 Win32 窗口处理了，对应的就是以上代码的等待渲染线程挂载窗口的一次渲染逻辑。这里需要说明的是上文说的等待渲染线程挂载窗口的一次渲染逻辑是我的猜测，根据 wpfgfx_cor3.dll 以及 SyncFlush 方法进行猜测的

在 EnsureHandle 之前设置 WindowStyle 则可以规避以上路径从而提升性能。提升性能的原因是在 EnsureHandle 之前，也就是 Win32 窗口创建之前，对 WindowStyle 的赋值走的是一个简单的属性赋值，毫无性能损耗。在真正创建窗口时，才读取 WindowStyle 属性，根据属性去创建 Win32 窗口。由于 WPF 应用本来就需要创建 Win32 窗口，也就是此时对 WindowStyle 的设置本身是不影响 Win32 窗口的创建的，换句话说就是在 Win32 窗口创建之前设置 WindowStyle 约等于免费

那在启动完成之后，窗口渲染完成之后设置 WindowStyle 呢？此时基本上不需要等待渲染，设置 WindowStyle 也就是一个 Win32 函数调用的损耗，大概测量时间在 30ms 左右。也就是说只有在启动过程中，想要做性能优化，才需要关注 EnsureHandle 之前设置 WindowStyle 属性。本文以上测试由 [lsj](https://blog.sdlsj.net/ ) 提供


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。