
本文将记录一个 dotnet 的已知问题。当自己不小心在方法上不正确标记了 MethodImplAttribute 特性时，错误选择了 MethodImplOptions.InternalCall 参数，那将会在运行的过程在，在此类型被访问之前就抛出了 System.TypeLoadException 异常，错误信息是 Internal call method with non_NULL RVA 内容

<!--more-->


<!-- CreateTime:2023/11/19 11:21:05 -->

<!-- 发布 -->
<!-- 博客 -->

遇到这个错误时，是比较难定位到具体的问题的。首先异常信息里面最多只是带上了类型名，没有告诉咱具体是哪个方法调用错误了。其次，异常的信息是 System.TypeLoadException 异常，且异常内容里面没有十分明确说明具体问题。不过有了 Internal call method 关键词倒是还能根据此找到问题

> 更新：在 2023.12.16 官方已经修复此问题，将会有更加明确的错误提示。预计跟随 dotnet 9 发布

下面我将和大家演示一下错误在方法上标记了 MethodImplOptions.InternalCall 特性参数的行为，以下的代码可以在本文末尾找到下载方法

```csharp
using System.Runtime.CompilerServices;

var f1 = new F1();

Console.WriteLine("Hello, World!");

class F1
{
    public F1()
    {
        var f2 = new F2();
    }
}

class F2
{
    [MethodImplAttribute(MethodImplOptions.InternalCall)]
    public void Foo()
    {

    }
}
```

上面代码之所以需要定义两个类型，是因为这个异常是会在 F2 类型被访问到之前就抛出异常，这也就导致了更加难以调试。上面代码运行的时候，异常是抛在了进入 F1 类型的构造函数里面，如下图

<!-- ![](image/dotnet 已知问题 错误标记 MethodImplOptions.InternalCall 特性参数将会在类型访问之前抛出 TypeLoadException 异常/dotnet 已知问题 错误标记 MethodImplOptions.InternalCall 特性参数将会在类型访问之前抛出 TypeLoadException 异常0.png) -->

![](http://cdn.lindexi.site/lindexi%2F202311191122288520.jpg)

这也就是导致了此问题更加难以调试的原因

在异常里面带上了 TypeName 属性，属性里面将会写明是 F2 类型出错，但是具体是哪个方法标记错了也没有更多的提示

我将此调试问题报告给 dotnet 官方，且官方已修复，请看 https://github.com/dotnet/runtime/issues/94967

> 更新： 现在可以拿到具体哪个类型的哪个方法标记错此特性

如果大家遇到了 System.TypeLoadException:“Internal call method with non_NULL RVA.” 异常，可以先看看这个异常里面的 TypeName 属性，确定是哪个类型出错了，然后再看看是否这个类型存在方法错误标记了 MethodImplOptions.InternalCall 特性参数导致运行失败

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/96014b44231c7e6920abf4373521359705e8cb0c/GagageheLoqearrergi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/96014b44231c7e6920abf4373521359705e8cb0c/GagageheLoqearrergi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 96014b44231c7e6920abf4373521359705e8cb0c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 96014b44231c7e6920abf4373521359705e8cb0c
```

获取代码之后，进入 GagageheLoqearrergi 文件夹


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。