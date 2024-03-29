
通过在 WPF 项目上安装 WindowsAppSDK 库，可以让 WPF 使用上 Win10 及以上版本提供的 Windows Runtime 强大的 API 集和使用上更多的黑科技。本文记录在安装 WindowsAppSDK 库之后，项目生成失败，提示 NETSDK1082 和 NETSDK1112 错误的问题，和修复的方法

<!--more-->


<!-- CreateTime:2022/10/25 10:23:06 -->

<!-- 发布 -->
<!-- 博客 -->

给 WPF 安装上 WindowsAppSDK 库，可以编辑 csproj 项目文件，添加如下代码进行安装

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.1.5" />
  </ItemGroup>
```

安装完成之后，需要给定的需要支持的系统版本，和对应的平台。由于 WindowsAppSDK 是继承 WinUI 或者说是 UWP 的写法，没有提供 Any CPU 的方式，是需要进行平台区分的，这就是为什么需要给定对应的平台的原因。编辑后的 csproj 项目文件内容如下


```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Platform>x86;x64</Platform>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.1.5" />
  </ItemGroup>

</Project>
```

才是运行构建提示错误，错误代码大概如下

```
1>C:\Program Files\dotnet\sdk\6.0.402\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.FrameworkReferenceResolution.targets(430,5): error NETSDK1082: There was no runtime pack for Microsoft.WindowsDesktop.App.WPF available for the specified RuntimeIdentifier 'win10-arm'.
1>C:\Program Files\dotnet\sdk\6.0.402\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.FrameworkReferenceResolution.targets(430,5): error NETSDK1082: There was no runtime pack for Microsoft.WindowsDesktop.App.WPF available for the specified RuntimeIdentifier 'win10-arm-aot'.
1>C:\Program Files\dotnet\sdk\6.0.402\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.FrameworkReferenceResolution.targets(430,5): error NETSDK1112: The runtime pack for Microsoft.Windows.SDK.NET.Ref was not downloaded. Try running a NuGet restore with the RuntimeIdentifier 'any'.
```

这是因为没有给定 RuntimeIdentifiers 从而才会提示 `error NETSDK1082: There was no runtime pack for Microsoft.WindowsDesktop.App.WPF available for the specified RuntimeIdentifier 'win10-arm'` 和 `error NETSDK1082: There was no runtime pack for Microsoft.WindowsDesktop.App.WPF available for the specified RuntimeIdentifier 'win10-arm-aot'` 错误的。因为本身 WPF 就不支持 `win10-arm` 和 `win10-arm-aot` 版本

修复方法是在 csproj 项目文件添加 RuntimeIdentifiers 的定义，如以下代码

```xml
  <RuntimeIdentifiers>win10-x86;win10-x64;win10-arm64</RuntimeIdentifiers>
```

由于我没有 win10-arm64 版本的系统，我就只采用 x86 和 x64 的。然后再加上能支持的最小系统版本，如 10.0.17763.0 版本

完成编辑的 csproj 项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Platform>x86;x64</Platform>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
    <RuntimeIdentifiers>win10-x86;win10-x64;win10-arm64</RuntimeIdentifiers>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.1.5" />
  </ItemGroup>

</Project>
```


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。