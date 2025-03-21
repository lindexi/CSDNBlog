
本文记录一个我在 git 钩子唤起一个 C# dotnet 的进程，在此进程里面使用 Process.Start 执行 git 命令的时候，被 git 钩子环境变量投毒的问题

<!--more-->


<!-- CreateTime:2025/03/15 07:15:26 -->

<!-- 发布 -->
<!-- 博客 -->

核心代码非常简单，我只是使用 git add 命令而已

```csharp
    var sourceFolder = @"C:\lindexi\Work\Source\";

            var processStartInfo = new ProcessStartInfo("git")
            {
                ArgumentList =
                {
                    "add",
                    "."
                },
                WorkingDirectory = sourceFolder,
            };

            Process.Start(processStartInfo)!.WaitForExit();
```

在 git 钩子里面调起进程的时候，会额外注入许多环境变量，比如 GIT_INDEX_FILE 和 GIT_DIR 等，这些环境变量是指定的是当前的 git 钩子所在的 git 仓库的

这就导致了我设置了 `WorkingDirectory = sourceFolder` 无效，直接导致 git add 内容到当前 git 钩子所在的仓库，而不是 `sourceFolder` 仓库

解决方法是清空环境变量，解决被 git 钩子投毒

```csharp
            var processStartInfo = new ProcessStartInfo("git")
            {
                ArgumentList =
                {
                    "add",
                    "."
                },
                WorkingDirectory = sourceFolder,
            };
            // 这是在 git 里面调用的，会被注入 git 的环境变量，从而被投毒，如 GIT_INDEX_FILE GIT_DIR 等，导致加入的文件不是在要求的路径
            processStartInfo.Environment.Clear();

            Process.Start(processStartInfo)!.WaitForExit();
```

以上代码核心是使用 `processStartInfo.Environment.Clear();` 清理所有传入到启动进程的环境变量

参考文档：

[Git - Git 钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。