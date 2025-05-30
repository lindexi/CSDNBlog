
本文告诉大家如何在源代码生成器 SourceGenerator 里面获取代码文件的本地文件的绝对路径

<!--more-->


<!-- CreateTime:2025/02/09 07:13:47 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

从 compilation 的 Options 拿到 SourceReferenceResolver 对象，调用其 NormalizePath 方法，传入 SyntaxTree 的 FilePath 参数即可

正常项目的 SourceReferenceResolver 都是存在的，尽管 Options 的 SourceReferenceResolver 被标记为可空。但从单元测试等进来的，则取决于单元测试的写法，可能为空

如果无法将传入的 FilePath 转换为本地的绝对路径，则会返回 null 的值

演示代码如下

```csharp
[Generator(LanguageNames.CSharp)]
public class IncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        IncrementalValuesProvider<GeneratorSyntaxContext> provider = context.SyntaxProvider.CreateSyntaxProvider((node, _) =>
        {
            return node.IsKind(SyntaxKind.ClassDeclaration);
        }, (syntaxContext, token) =>
        {
            return syntaxContext;
        });

        var pathProvider = provider.Combine(context.CompilationProvider).Select((tuple, _) =>
        {
            var (syntaxContext, compilation) = tuple;

            var path = compilation.Options.SourceReferenceResolver?.NormalizePath(syntaxContext.Node.SyntaxTree.FilePath,
                baseFilePath: null);
            // 找不到返回 path 为 null 的值
            return path;
        });

        context.RegisterSourceOutput(pathProvider, (productionContext, path) =>
        {
        });
    }
}
```

单元测试里面取决于 CSharpCompilationOptions 是否传入 sourceReferenceResolver 参数决定 Options 的 SourceReferenceResolver 是否是空。如以下代码就传入了 SourceFileResolver 对象，设置了代码查询路径

```csharp
    private static CSharpCompilation CreateCompilation(string source)
        => CSharpCompilation.Create(...,
            new CSharpCompilationOptions(OutputKind.ConsoleApplication, sourceReferenceResolver: new SourceFileResolver([], @"C:\lindexi\Code")));
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0b5550ce0e2736df6f2aac01f1f65ca37103fbdf/Workbench/KonanohallreGonurliyage) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0b5550ce0e2736df6f2aac01f1f65ca37103fbdf/Workbench/KonanohallreGonurliyage) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0b5550ce0e2736df6f2aac01f1f65ca37103fbdf
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0b5550ce0e2736df6f2aac01f1f65ca37103fbdf
```

获取代码之后，进入 Workbench/KonanohallreGonurliyage 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。