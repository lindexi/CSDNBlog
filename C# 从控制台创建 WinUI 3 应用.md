
本文将告诉大家如何从控制台而不是 WinUI3 模版项目，从零一步步创建出 WinUI 3 应用

<!--more-->


<!-- CreateTime:2023/7/31 16:35:10 -->

<!-- 发布 -->
<!-- 博客 -->

本文不是 WinUI 3 入门博客，本文将从比较基础层的方式创建出 WinUI 3 应用，适合于了解 WinUI 3 的伙伴阅读。阅读本文可以更好理解 WinUI 3 应用项目的初始化逻辑

在开始之前，先新建一个 dotnet 6 控制台项目，接着编辑项目的 csproj 项目文件，替换为如下代码进行初始化

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PlatformTarget>x86</PlatformTarget>
    <RuntimeIdentifiers>win10-x86;win10-x64</RuntimeIdentifiers>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <UseWinUI>true</UseWinUI>
    <WindowsPackageType>None</WindowsPackageType>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.3.230331000" />
  </ItemGroup>


</Project>
```

为了减少细节的干扰，本文这里采用的是引用 Microsoft.WindowsAppSDK 的方式而不是自己编写所有与 COM 连接的基础代码。如果对细节逻辑好奇的话，推荐阅读开源的 Microsoft.WindowsAppSDK 的源代码： [https://github.com/microsoft/windowsAppSDK](https://github.com/microsoft/windowsAppSDK)

如果自己编写所有和 COM 连接的基础代码`，以上的 csproj 的 `net6.0-windows10.0.19041` 等限制是可以去掉的

接下来进入到 Program.cs 文件里面，咱将在这个文件里面编写创建 WinUI 3 应用的代码。在开始之前，还需要编写一个辅助类型，用于方便编写出代码，辅助类型的代码如下

```csharp
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;

public class App : Application
{
    public event EventHandler<LaunchActivatedEventArgs>? Launched;

    protected override void OnLaunched(LaunchActivatedEventArgs args)
    {
        Launched?.Invoke(this, args);
    }
}
```

进入到 Main 函数的第一句话就是调用 ComWrappersSupport 的 InitializeComWrappers 方法进行 WinUI 3 的 COM 调用的连接初始化，代码如下

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        global::WinRT.ComWrappersSupport.InitializeComWrappers();
    }
```

和 WPF 不相同的是 WinUI 3 是没有要求在 Main 函数采用 STA 线程的，也就是直接编写调用代码即可

接下来是调用 Microsoft.UI.Xaml.Application 的 Start 方法启动应用，如以下代码

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        global::WinRT.ComWrappersSupport.InitializeComWrappers();
        global::Microsoft.UI.Xaml.Application.Start((p) =>
        {
        });
    }
}
```

不严谨的说，这么简单的逻辑就跑起来了 WinUI 3 应用了。如果还想看到界面，那就需要额外创建窗口，如下面代码

```csharp
        global::Microsoft.UI.Xaml.Application.Start((p) =>
        {
            var app = new App();
            app.Launched += (sender, e) =>
            {
                var window = new Window()
                {
                    Title = "控制台创建应用"
                };
                window.Content = new Grid()
                {
                    Children =
                    {
                        new TextBlock()
                        {
                            Text = "控制台应用",
                            HorizontalAlignment = HorizontalAlignment.Center,
                            VerticalAlignment = VerticalAlignment.Center
                        }
                    }
                };
                window.Activate();
            };
        });
```

以上代码创建了 App 类型，创建出来之后，将会由在 COM 层实现的底层调用到 OnLaunched 方法，然后到 Launched 事件。创建了窗口，调用 Activate 让窗口显示

全部代码如下

```csharp
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;

namespace FinayfuweewawWakibawlu;

public class App : Application
{
    public event EventHandler<LaunchActivatedEventArgs>? Launched;

    protected override void OnLaunched(LaunchActivatedEventArgs args)
    {
        Launched?.Invoke(this, args);
    }
}

internal class Program
{
    static void Main(string[] args)
    {
        global::WinRT.ComWrappersSupport.InitializeComWrappers();
        global::Microsoft.UI.Xaml.Application.Start((p) =>
        {
            var app = new App();
            app.Launched += (sender, e) =>
            {
                var window = new Window()
                {
                    Title = "控制台创建应用"
                };
                window.Content = new Grid()
                {
                    Children =
                    {
                        new TextBlock()
                        {
                            Text = "控制台应用",
                            HorizontalAlignment = HorizontalAlignment.Center,
                            VerticalAlignment = VerticalAlignment.Center
                        }
                    }
                };
                window.Activate();
            };
        });
    }
}
```

通过以上的代码即可在控制台创建 WinUI 3 应用


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。