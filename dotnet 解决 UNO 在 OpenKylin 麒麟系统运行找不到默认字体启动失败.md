
本文记录 UNO 应用在 OpenKylin 麒麟系统运行找不到默认字体启动失败的解决方法

<!--more-->


<!-- CreateTime:2024/06/26 07:25:47 -->

<!-- 发布 -->
<!-- 博客 -->

本文方法适用于 5.2.161 版本的 UNO 应用，更高版本我没有经过充分测试

在 OpenKylin 系统启动 Avalonia 应用失败，在 FontDetailsCache 的 GetFontInternal 方法里面抛出空异常

核心原因是 UNO 尝试寻找默认的字体，然而找不到

解决方法是手动设置默认字体，编辑 `App.xaml.cs` 加上如下代码

```csharp
    public App()
    {
        this.InitializeComponent();
#if HAS_UNO
        // Fix run fail in Kylin system.
        // https://github.com/unoplatform/uno/issues/17287
        FeatureConfiguration.Font.DefaultTextFontFamily = "Noto Sans CJK SC";
#endif
    }
```

此问题已经报告给 UNO 官方，请看 : <https://github.com/unoplatform/uno/issues/17287>

相关博客： [dotnet 解决 Avalonia 在 OpenKylin 麒麟系统运行找不到默认字体启动失败](https://blog.lindexi.com/post/dotnet-%E8%A7%A3%E5%86%B3-Avalonia-%E5%9C%A8-OpenKylin-%E9%BA%92%E9%BA%9F%E7%B3%BB%E7%BB%9F%E8%BF%90%E8%A1%8C%E6%89%BE%E4%B8%8D%E5%88%B0%E9%BB%98%E8%AE%A4%E5%AD%97%E4%BD%93%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5.html )

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/bf72c0981965bb4aed7f082ef81eed2e4b6d6913/UnoDemo/DalekairwiJebonacaki) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bf72c0981965bb4aed7f082ef81eed2e4b6d6913/UnoDemo/DalekairwiJebonacaki) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bf72c0981965bb4aed7f082ef81eed2e4b6d6913
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bf72c0981965bb4aed7f082ef81eed2e4b6d6913
```

获取代码之后，进入 UnoDemo/DalekairwiJebonacaki 文件夹


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。