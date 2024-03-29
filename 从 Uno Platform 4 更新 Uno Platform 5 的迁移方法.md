
本文记录我的一个小项目从 Uno Platform 4 更新 Uno Platform 5 的一些变更和迁移方法，由于项目太小，可能踩到的坑不多

<!--more-->


<!-- CreateTime:2024/1/17 14:39:36 -->
<!-- 发布 -->
<!-- 博客 -->

官方文档： [Migrating to Uno Platform 5.0](https://platform.uno/docs/articles/migrating-to-uno-5.html )

表扬一下官方，文档写的很详细

## Directory.Packages.props

所有的 Uno 相关包的版本更新：

```diff
-    <PackageVersion Include="Uno.WinUI" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Lottie" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Lottie" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.RemoteControl" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.RemoteControl" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Gtk" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Gtk" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Linux.FrameBuffer" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Linux.FrameBuffer" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Wpf" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Wpf" Version="5.0.143" />
```

## Skia.WPF

这里有大改的部分，不仅仅只是修改命名空间

之前的 WpfHost 是放入到具体的 WPF 的窗口里面的，现在更新版本是直接放入到 App 级的。迁移步骤如下

- 在 `App.xaml` 文件里面删除 `StartupUri="Wpf/MainWindow.xaml"` 属性的配置
- 删除 `MainWindow.xaml` 和 `MainWindow.xaml.cs` 文件
- 在 `App.xaml.cs` 里添加以下代码到构造函数

```csharp
public App()
{
    var host = new WpfHost(Dispatcher, () => new AppHead());
    host.Run();
}
```

如果没有更改，将会在运行看到 `If you are seeing this, make sure to follow the "Migrating WpfHost" section of Migrating from previous releases article in the Uno Platform documentation at https://aka.platform.uno/uno5-wpfhost-migration. WpfHost is used at the application level instead of window level starting Uno Platform 5.0.` 提示

## Skia.Framebuffer

更换了命名空间，从原本的 `Uno.UI.Runtime.Skia.FrameBufferHost` 变更为 `Uno.UI.Runtime.Skia.Linux.FrameBuffer.FrameBufferHost` 命名空间下的类型

只需加上 `using Uno.UI.Runtime.Skia.Linux.FrameBuffer;` 即可

## Skia.Gtk

更换了命名空间，从原本的 `Uno.UI.Runtime.Skia.GtkHost` 变更为 `Uno.UI.Runtime.Skia.Gtk.GtkHost` 命名空间下的类型

只需加上 `using Uno.UI.Runtime.Skia.Gtk;` 即可


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。