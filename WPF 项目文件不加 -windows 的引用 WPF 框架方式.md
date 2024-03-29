
默认情况下的 WPF 项目都是带 -windows 的 TargetFramework 方式，但有一些项目是不期望加上 -windows 做平台限制的，本文将介绍如何实现不添加 -windows 而引用 WPF 框架

<!--more-->


<!-- CreateTime:2023/7/17 19:46:06 -->

<!-- 博客 -->
<!-- 发布 -->

先看一个标准的 WPF 项目的 csproj 项目文件内容

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

</Project>
```

以上代码的核心在于设置 TargetFramework 为 `net7.0-windows` 的同时设置 `UseWPF` 属性。在此设置之下，项目本身就限定了采用 Windows 平台

对于一些特殊的项目来说，也许只是在某些模块下期望引用 WPF 的某些类型，而不想自己的项目限定平台。这时候可以去掉 `-windows` 换成 FrameworkReference 的方式，如以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.WindowsDesktop.App" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />
  </ItemGroup>

</Project>
```

通过 `<FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />` 即可设置对 WPF 程序集的引用，也就是仅仅只是将 WPF 的程序集取出来当成引用，而不是加上 WPF 的负载

通过此方式即可不需要设置 TargetFramework 为 `net7.0-windows` 和设置 `UseWPF` 属性

以上方法对于 WinForms 项目来说也是一样的，如果也需要加上 WinForms 程序集的引用，可以添加 `<FrameworkReference Include="Microsoft.WindowsDesktop.App.WindowsForms" />` 如代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.WindowsDesktop.App" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WindowsForms" />
  </ItemGroup>

</Project>
```

以上方法对 .NET 7 或 .NET 6 都生效



我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。