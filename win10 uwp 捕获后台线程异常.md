
本文告诉大家如何在 UWP 中捕获全局的后台线程异常，在出现后台线程异常时，将会让 UWP 程序闪退，但是在退出之前还是可以执行自己的代码

<!--more-->


<!-- CreateTime:2019/12/16 8:56:13 -->

<!-- 发布 -->

在 UWP 中，如果需要捕获前台线程，也就是 UI 线程的异常，可以参见 [UWP 中的全局异常处理](https://www.cnblogs.com/zhs852/p/uwp-global-error-handler.html) 的方法

在 App 的构造函数添加 UnhandledException 事件，在事件方法里面通过参数 UnhandledExceptionEventArgs 可以设置当前这个异常是否被处理，如设置为 true 那么就是被处理的异常，此时的应用不会闪退

```csharp
        public App()
        {
            this.InitializeComponent();
            this.Suspending += OnSuspending;
            UnhandledException += App_UnhandledException;
        }

        private void App_UnhandledException(object sender, Windows.UI.Xaml.UnhandledExceptionEventArgs e)
        {
            e.Handled = true;// 设置为 true 那么表示这个异常被处理，应用不会闪退
        }
```

如果是后台线程异常，需要使用 AppDomain.CurrentDomain.UnhandledException 事件

```csharp
        public App()
        {
            this.InitializeComponent();
            this.Suspending += OnSuspending;
            AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;
        }

        private void CurrentDomain_UnhandledException(object sender, System.UnhandledExceptionEventArgs e)
        {
            // 后台线程异常，执行到这里的应用就会闪退
        }
```

触发后台线程异常很简单，请看下面代码

```csharp
            var thread = new Thread(() => throw new Exception());
            thread.Start();
```

执行到创建线程然后在线程抛出异常，将会进入 `CurrentDomain_UnhandledException` 方法，然后应用程序退出。通过这个方法可以在软件退出前做日志记录

[Application.UnhandledException Event (Windows.UI.Xaml) ](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.application.unhandledexception)

[UWP 中的全局异常处理](https://www.cnblogs.com/zhs852/p/uwp-global-error-handler.html)



我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

各大城市俱乐部QQ群

- 广州.NET俱乐部 651477282
- 济南.NET俱乐部 456378675
- 上海.NET俱乐部 197239851
- 苏州.NET俱乐部 953297025
- 厦门.NET俱乐部 57043161
- 福州.NET俱乐部 710090446
- 西安.NET俱乐部 227283081
- 青岛.NET俱乐部 34721773

更多城市.NET俱乐部请在QQ群上搜

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。