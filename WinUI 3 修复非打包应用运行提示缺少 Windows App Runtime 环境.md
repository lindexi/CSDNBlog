
本文将告诉大家如何修复 WinUI 3 非打包的应用，在分发给到客户时，在客户的机器上运行提示缺少 Windows App Runtime 环境

<!--more-->


<!-- CreateTime:2023/6/15 19:31:03 -->


<!-- 发布 -->
<!-- 博客 -->

在用户的机器上提示的 Windows App Runtime 环境缺失的提示内容大概如下

> This application requires the Windows App Runtime
>
> Version 1.3
>
> (MSIX package version >= 3000.882.2207.0)
>
> Do you want to install a compatible Windows App Runtime now?

<!-- ![](image/WinUI 3 修复非打包应用运行提示缺少 Windows App Runtime 环境/WinUI 3 修复非打包应用运行提示缺少 Windows App Runtime 环境0.png) -->
![](http://cdn.lindexi.site/lindexi%2F20236151933384613.jpg)

点击确定时，默认跳转到 [https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads) 下载页

这时可选下载安装 Windows App Runtime 安装包 WindowsAppRuntimeInstall.exe 文件修复此问题

当然了，这样的应用要是真就需要用户去额外安装 WindowsAppRuntimeInstall.exe 文件，肯定是不成的。本文告诉大家如何做到让 WinUI 3 应用和传统的 Win32 应用一样，拷贝过去到用户的机器上就能运行

第一步，保持让 Microsoft.WindowsAppSDK 这个 NuGet 包在 1.2 以上的版本。当前是 2023.6.15 最新版本是 1.3.230602002 版本。因为根据 [官方文档](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/self-contained-deploy/deploy-self-contained-apps) 可以了解到，在 1.2 版本才加上了 `WindowsAppSdkUndockedRegFreeWinRTInitialize` 功能。想要不弹出以上的提示框，最好是需要有 `WindowsAppSdkUndockedRegFreeWinRTInitialize` 功能

第二步，配置 csproj 项目文件，设置 WindowsAppSDKSelfContained 为 true 和设置 WindowsPackageType 为 None 值。默认情况下，将会在 WindowsAppSDKSelfContained 为 true 和 WindowsPackageType 属性为 None 值时，自动在 SDK 里面配置 WindowsAppSdkUndockedRegFreeWinRTInitialize 为 true 值

修改之后的 csproj 项目文件大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWinUI>true</UseWinUI>
    <Platforms>x86;x64</Platforms>
    <RuntimeIdentifiers>win10-x86;win10-x64</RuntimeIdentifiers>

    <!-- 下面两句代码是核心 -->
    <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
    <WindowsPackageType>None</WindowsPackageType>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.3.230602002" />
    <PackageReference Include="Microsoft.Windows.SDK.BuildTools" Version="10.0.22621.756" />
  </ItemGroup>
</Project>
```

通过以上方式修改之后，即可进行独立发布后，应用无需安装 Windows App Runtime 环境，双击即可运行

由于目前关于 WindowsAppSdkUndockedRegFreeWinRTInitialize 和 UndockedRegFreeWinRT 的官方文档较为缺乏，我对这部分知识的细节并不了解，如果您对此感兴趣，请自行了解

---

补充：

调用 `global::Microsoft.UI.Xaml.Application.Start` 方法时，抛出 没有注册类 (0x80040154 (REGDB_E_CLASSNOTREG)) 异常，如以下代码

```csharp
            global::WinRT.ComWrappersSupport.InitializeComWrappers();

            // 如果不在 csproj 加上
            // <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
            // <WindowsPackageType>None</WindowsPackageType>
            // 将抛出 没有注册类 (0x80040154 (REGDB_E_CLASSNOTREG))
            (p =>
            {
            });
```

解决方法也是在 csproj 上配置 WindowsAppSDKSelfContained 和 WindowsPackageType 属性

配置之后的 csproj 大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows10.0.22000</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <PlatformTarget>x86</PlatformTarget>
    <Platform>x86</Platform>
    <RuntimeIdentifier>win-x86</RuntimeIdentifier>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
    <WindowsSdkPackageVersion>10.0.22000.38</WindowsSdkPackageVersion>

    <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
    <WindowsPackageType>None</WindowsPackageType>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.6.241114003" />
  </ItemGroup>
  
</Project>
```

和上文的 `net6.0-windows10.0.19041` 的 csproj 不同的是，以上的 csproj 只写了 x86 平台


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。