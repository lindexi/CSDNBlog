
本文记录 WPF 的已知问题，在 .NET Core 版本的 WPF 应用里面，应用启动的过程监听 WMI 事件，将导致触摸模块 COM 接口获取失败，进而导致触摸失效

<!--more-->


<!-- CreateTime:2024/09/11 07:19:40 -->

<!-- 发布 -->
<!-- 博客 -->

此问题仅在 .NET Core 版本复现，在 .NET Framework 框架下能正常工作

复现步骤如下：

1. 安装 System.Management 库用于使用 WqlEventQuery 监听 WMI 变更
2. 监听 TouchDown 事件输出断点信息

复现代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        AppDomain.CurrentDomain.FirstChanceException += (sender, args) =>
        {
            Debug.WriteLine(args.Exception);
        };

        WqlEventQuery insertQuery =
            new WqlEventQuery("SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE TargetInstance ISA 'Win32_USBHub'");
        ManagementEventWatcher insertWatcher = new ManagementEventWatcher(insertQuery);
        insertWatcher.Start(); // 如果注释掉这句话，那 TouchDown 事件将会被触发

        TouchDown += MainWindow_TouchDown;
    }

    private void MainWindow_TouchDown(object? sender, TouchEventArgs e)
    {
        Debugger.Break(); // 不会被命中
    }
```

加上 `insertWatcher.Start()` 这句代码时，可以从 FirstChanceException 看到如下异常

```
System.InvalidCastException: 没有注册接口


   at MS.Win32.Penimc.UnsafeNativeMethods.CoCreateInstance(Guid& clsid, Object punkOuter, Int32 context, Guid& iid)
```

规避方法是不要在软件启动的时候立刻调用 WMI 组件，如让 ManagementEventWatcher 延迟几秒执行

此问题已经报告给 WPF 官方，请看 <https://github.com/dotnet/wpf/issues/9752>

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/308096e0c8ede019f6dbe5bfe974ae1a12d7de42/WPFDemo/YanerehaylemJeekalhebel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/308096e0c8ede019f6dbe5bfe974ae1a12d7de42/WPFDemo/YanerehaylemJeekalhebel) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 308096e0c8ede019f6dbe5bfe974ae1a12d7de42
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 308096e0c8ede019f6dbe5bfe974ae1a12d7de42
```

获取代码之后，进入 WPFDemo/YanerehaylemJeekalhebel 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。