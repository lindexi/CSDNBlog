
本文将告诉大家如何使用 ForAttributeWithMetadataName 方法用来提高 IIncrementalGenerator 增量 Source Generator 源代码生成的开发效率以及提高源代码生成器的运行效率

<!--more-->


<!-- CreateTime:2024/2/5 17:57:13 -->

<!-- 发布 -->
<!-- 博客 -->

这是一个在 2022 的 6 月 15 才合入的新功能。原因是 Roslyn 团队发现了大量的源代码生成器和分析器项目都十分依赖 Attribute 的判断，且许多团队在实现的过程中都很难实现正确的增量方式，导致了许多多余浪费的计算，影响性能

使用 ForAttributeWithMetadataName 的方法非常简单，和 CreateSyntaxProvider 方法非常相似，只是添加了多了一个参数，这个参数就是所期望的 Attribute 名。假定正在编写的源代码生成器或分析器强依赖某个已知的特性，那通过 ForAttributeWithMetadataName 方法即可减少一些重复代码的编写和提升性能

代码例子如下

```csharp
        var provider = context.SyntaxProvider.ForAttributeWithMetadataName("Lindexi.FooAttribute",
            // 进一步判断
            (node, _) => node.IsKind(SyntaxKind.ClassDeclaration),
            (syntaxContext, _) => syntaxContext.TargetSymbol.Name);
```

通过 ForAttributeWithMetadataName 方法，第一个参数传入所期望的特性名，第二个参数和第三个参数将和 CreateSyntaxProvider 的第一个第二个参数相同，只是传入的数据有所差别。此时的第二个参数的委托的 SyntaxNode 类型的对象就是已经标记了第一个参数的特性的成员，一般的进一步判断的代码都能简单，除非是有业务需要的过滤条件

以上的代码是过滤出所有标记了 `Lindexi.FooAttribute` 特性的类型

通过 ForAttributeWithMetadataName 方法可以由 Roslyn 底层尽量保持增量执行，也就是没有变更的情况下不会执行，可以很大提升性能

可以试试将这些类型添加到源代码生成里面。详细的代码可以通过下文获取所有的代码，获取的代码里面还包含了源代码生成的单元测试，可以运行单元测试内容了解具体的生成器输出以及进行调试

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/ed3de9582fa11bb0c9b0fd02e34d2c4f89c21df0/JijalnairhelecheNakallchijall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ed3de9582fa11bb0c9b0fd02e34d2c4f89c21df0/JijalnairhelecheNakallchijall) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ed3de9582fa11bb0c9b0fd02e34d2c4f89c21df0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ed3de9582fa11bb0c9b0fd02e34d2c4f89c21df0
```

获取代码之后，进入 JijalnairhelecheNakallchijall 文件夹

更多关于源代码生成博客请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。