
在我所在的团队开发的一个 WPF 应用程序里面，使用到了 SQLite 作为本地数据库。在优化启动性能过程中，发现了在启动过程一旦访问 SQLite 将会因为 SQLiteFunction 扫描程序集导致 CPU 损耗，从而影响启动性能。本文将告诉大家如何禁用 SQLite 的 SQLiteFunction 扫描程序集

<!--more-->


<!-- CreateTime:2024/2/20 10:26:01 -->

<!-- 发布 -->
<!-- 博客 -->

在 SQLiteFunction 模块里面，将会扫描全部程序集，用来找到 SQLiteFunctionAttribute 实现类型收集。刚好我的应用不需要这样的功能，这就意味着在 SQLiteFunction 模块里面扫描全部程序集的逻辑是白跑的，禁用此逻辑可提升启动性能

禁用方法可以是在 Main 方法里面设置环境变量的方式实现禁用 SQLiteFunction 模块扫描全部程序集。以下代码需要放在应用程序运行足够早的时间，至少需要比首个 SQLite 相关模块早，由于以下代码仅仅只是设置环境变量，性能损耗上很小，可以比较放心的在 Main 方法里面设置

```csharp
       // 在 SQLite 的 SQLiteFunction 类的静态构造函数会反射扫一遍所有程序集，找 SQLiteFunctionAttribute 特性
       // 我们不需要这个功能，通过配置这个环境变量，避免扫描，优化启动时 CPU 占用
       Environment.SetEnvironmentVariable("No_SQLiteFunctions", "1");
```

特别感谢 [lsj](https://blog.sdlsj.net/ ) 提供的方法，我只是代为记录

以上环境变量请参阅 [SQLite Environment Variables](https://system.data.sqlite.org/index.html/doc/216889c23b/Doc/Extra/Provider/environment.html ) 官方文档


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。