
在自定义 WPF 控件库时，可以看到有一些控件会加上 DefaultEventAttribute 特性，通过这个特性可以告诉 XAML 编辑器，默认创建的事件是什么

<!--more-->



<!-- 发布 -->

如编写一个 UserControl1 的用户自定义控件，可以在 xaml.cs 代码加上此特性

```csharp
    [DefaultEvent("Foo")]
    public partial class UserControl1
    {
        public event EventHandler Foo;
    }
```

在 MainWindow 上添加 UserControl1 然后构建一下代码

```csharp
<local:UserControl1 />
```

在 XAML 编辑器双击一下 UserControl1 控件，可以看到自动生成如下代码

```csharp
<local:UserControl1 Foo="UserControl1_Foo"/>

        private void UserControl1_Foo(object sender, EventArgs e)
        {

        }
```



我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。