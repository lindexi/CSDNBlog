
本文将告诉大家如何在 IIncrementalGenerator 增量 Source Generator 生成代码里面，在 Roslyn 分析器里面判断两个程序集是否存在引用关系

<!--more-->


<!-- CreateTime:2023/8/23 10:03:24 -->
<!-- 标题： IIncrementalGenerator 判断程序集的引用关系 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

先上核心代码实现，核心代码就是调用以下代码进行判断程序集的引用关系

```csharp
        private static bool IsReference(IAssemblySymbol currentAssemblySymbol, IAssemblySymbol requiredAssemblySymbol,
            Dictionary<IAssemblySymbol, bool /*是否引用*/> visited)
        {
            if (SymbolEqualityComparer.Default.Equals(currentAssemblySymbol, requiredAssemblySymbol))
            {
                // 这个就看业务了，如果两个程序集是相同的，是否判断为引用关系
                return true;
            }

            foreach (var moduleSymbol in currentAssemblySymbol.Modules)
            {
                foreach (var referencedAssemblySymbol in moduleSymbol.ReferencedAssemblySymbols)
                {
                    if (SymbolEqualityComparer.Default.Equals(referencedAssemblySymbol, requiredAssemblySymbol))
                    {
                        // 记录当前程序集存在引用关系
                        visited[currentAssemblySymbol] = true;
                        return true;
                    }
                    else
                    {
                        if (visited.TryGetValue(referencedAssemblySymbol, out var isReference))
                        {
                            // 这个是访问过的，那就从字典获取缓存，不需要再访问一次
                            // 同时也能解决程序集循环引用问题
                        }
                        else
                        {
                            // 没有访问过的，获取引用的程序集是否存在引用关系
                            isReference = IsReference(referencedAssemblySymbol, requiredAssemblySymbol, visited);
                            visited[referencedAssemblySymbol] = isReference;
                        }

                        if (isReference)
                        {
                            // 如果这个程序集有引用，那也算上
                            return true;
                        }
                    }
                }
            }

            return false;
        }
```

核心判断逻辑就是获取当前程序集的 Modules 从而获取到 ReferencedAssemblySymbols 也就是当前程序集所引用的所有程序集，再判断引用的程序集里面是否包含期望被引用的程序集

对于间接引用的程序集来说，需要进行遍历才可以判断到，但是此时需要处理程序集循环引用问题。本文这里采用传入一个字典的方法解决这个问题

获取当前分析器所分析的项目的引用程序集，可以通过 Compilation 的 SourceModule 的 ReferencedAssemblySymbols 拿到，如以下代码

```csharp
            context.CompilationProvider.Select((compilation, _) =>
            {
                var sourceModuleReferencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;
                ... // 忽略代码
            });
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c87c7e78b068956f3f8f4640a85c04a3af9e07fc/HulanucerbeljuChaijacemjarga) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c87c7e78b068956f3f8f4640a85c04a3af9e07fc/HulanucerbeljuChaijacemjarga) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c87c7e78b068956f3f8f4640a85c04a3af9e07fc
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c87c7e78b068956f3f8f4640a85c04a3af9e07fc
```

获取代码之后，进入 HulanucerbeljuChaijacemjarga 文件夹


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。