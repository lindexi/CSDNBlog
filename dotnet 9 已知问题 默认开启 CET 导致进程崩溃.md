
本文记录 dotnet 9 的一个已知且当前已修问题。默认开启 CET 导致一些模块执行时触发崩溃

<!--more-->


<!-- CreateTime:2025/02/06 07:13:36 -->

<!-- 发布 -->
<!-- 博客 -->

官方文档： [Breaking change: CET supported by default - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/compatibility/interop/9.0/cet-support )

表现：

调用 OpenFileDialog 的 ShowDialog 将会异常崩溃，崩溃异常是 `FAST_FAIL_SET_CONTEXT_DENIED`

或退出错误码为 0xc0000409 STATUS_STACK_BUFFER_OVERRUN

修复的代码：

此问题已在 <https://github.com/dotnet/runtime/pull/109074> 修复

永久解决方法：

更新 dotnet sdk 版本，再次构建发布

理论分析更新 dotnet runtime 也有用，但我没有尝试过，欢迎大佬补充

临时解决方法：

如果不便更新 dotnet sdk 版本，可通过如下设置禁用 CET 开启

```xml
<CETCompat>false</CETCompat>
```

以上设置放在 csproj 项目文件的 PropertyGroup 里面，更改之后的 csproj 代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <CETCompat>false</CETCompat>
  </PropertyGroup>

</Project>
```

相关问题：

- <https://github.com/dotnet/wpf/issues/10305>
- <https://github.com/dotnet/wpf/issues/10318>


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。