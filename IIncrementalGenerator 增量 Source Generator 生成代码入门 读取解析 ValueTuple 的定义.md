
本文将告诉大家如何在分析器里面解析代码里面对于 ValueTuple 的定义，包括如何获取 ValueTuple 里面的 Item 的类型和命名

<!--more-->


<!-- CreateTime:2023/8/21 19:51:17 -->
<!-- 标题： IIncrementalGenerator 解析 ValueTuple 的定义 -->

<!-- 发布 -->
<!-- 博客 -->

<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

开始之前先创建一个用来被分析的项目，在这个项目里面定义 Foo1 类型，然后再定义 F2 方法，设置 F2 方法的返回值是一个 ValueTuple 类型，如以下代码

```csharp
public class Foo1
{
    public (int x, int) F2()
    {
        return default;
    }
}
```

本文将使用此作为例子，告诉大家如何解析 ValueTuple 的定义，也就是获取 F2 方法返回值类型的定义

先编写语法过滤，只让方法定义的语法通过，如以下代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((node, _) =>
            {
                // 只读取方法定义。这里例子是读取一个方法的返回值
                return node.IsKind(SyntaxKind.MethodDeclaration);
            }, (syntaxContext, _) =>
            {
                ... // 忽略代码
            });

            ... // 忽略代码
        }
```

因为本文是使用分析 F2 方法的返回值作为例子，因此上面代码只让方法定义的语法通过

接下来获取方法的返回值，如以下代码

```csharp
            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((node, _) =>
            {
                // 只读取方法定义。这里例子是读取一个方法的返回值
                return node.IsKind(SyntaxKind.MethodDeclaration);
            }, (syntaxContext, _) =>
            {
                // 转换为语义
                IMethodSymbol methodSymbol =
                    syntaxContext.SemanticModel.GetDeclaredSymbol(syntaxContext.Node) as IMethodSymbol;
                if (methodSymbol is null)
                {
                    // 理论上不会进入这个代码，前面判断方法定义
                    return string.Empty;
                }

                // 获取方法返回类型
                ITypeSymbol methodSymbolReturnType = methodSymbol.ReturnType;
                if (methodSymbolReturnType is INamedTypeSymbol namedTypeSymbol)
                {
                }

                ... // 忽略代码

            }).Where(t => !string.IsNullOrEmpty(t));
```

拿到方法返回值，可以进行判断 ValueTuple 类型，判断方法是通过先判断是否值类型，再判断 Tuple 元素的数量

```csharp
                    // 由于 ValueTuple 是值类型，因此可以快速判断是否值类型
                    if (namedTypeSymbol.IsValueType && namedTypeSymbol.TupleElements.Length > 0)
                    {
                    }
```

获取 ValueTuple 语义上定义的各个 Item 的类型和命名可以通过遍历 TupleElements 属性

```csharp
                        foreach (var tupleElement in namedTypeSymbol.TupleElements)
                        {
                            // 如此可以获取每一个类型
                            var tupleElementType = tupleElement.Type;
                        }
```

如果用户代码里面没有给 Item 命名，将会自动填充默认的 Item1 等默认命名，如果想要获取原来代码的定义，可以获取语法内容，如以下代码

```csharp
                        var code = namedTypeSymbol.DeclaringSyntaxReferences[0].GetSyntax().ToString();
```

以下是 CodeCollectionIncrementalGenerator 的全部代码，可以创建出一个 Foo 类型，用来输出 Foo1 的 F2 方法的返回代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((node, _) =>
            {
                // 只读取方法定义。这里例子是读取一个方法的返回值
                return node.IsKind(SyntaxKind.MethodDeclaration);
            }, (syntaxContext, _) =>
            {
                // 转换为语义
                IMethodSymbol methodSymbol =
                    syntaxContext.SemanticModel.GetDeclaredSymbol(syntaxContext.Node) as IMethodSymbol;
                if (methodSymbol is null)
                {
                    // 理论上不会进入这个代码，前面判断方法定义
                    return string.Empty;
                }

                // 获取方法返回类型
                ITypeSymbol methodSymbolReturnType = methodSymbol.ReturnType;
                if (methodSymbolReturnType is INamedTypeSymbol namedTypeSymbol)
                {
                    // 由于 ValueTuple 是值类型，因此可以快速判断是否值类型
                    if (namedTypeSymbol.IsValueType && namedTypeSymbol.TupleElements.Length > 0)
                    {
                        foreach (var tupleElement in namedTypeSymbol.TupleElements)
                        {
                            // 如此可以获取每一个类型
                            var tupleElementType = tupleElement.Type;
                        }

                        var code = namedTypeSymbol.DeclaringSyntaxReferences[0].GetSyntax().ToString();
                        return code;
                    }
                }

                return "";
            }).Where(t => !string.IsNullOrEmpty(t));

            context.RegisterImplementationSourceOutput(incrementalValuesProvider,
                (productionContext, provider) =>
                {
                    var text = provider;

                    var code = @"using System;
namespace LainewihereJerejawwerye
{
    public static class Foo
    {
        public static void F1()
        {
            Console.WriteLine(""Foo1.F2 方法返回值是" + text + @""");
        }
    }
}";
                    productionContext.AddSource("Demo", code);
                });
        }
    }
}
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/ed913bc50dcdbdf67bd387cd49d0cfd2a95b4ede/LainewihereJerejawwerye) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ed913bc50dcdbdf67bd387cd49d0cfd2a95b4ede/LainewihereJerejawwerye) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ed913bc50dcdbdf67bd387cd49d0cfd2a95b4ede
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ed913bc50dcdbdf67bd387cd49d0cfd2a95b4ede
```

获取代码之后，进入 LainewihereJerejawwerye 文件夹

更多源代码生成，请看官方的 [Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md)

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。