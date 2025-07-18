
进行框架依赖发布的时候，应用程序需要有 dotnet runtime 运行时才能跑起来。在 dotnet 9 之前，通常都是需要安装到系统的 Program File 文件夹下的全局 dotnet 运行时的支持。在 dotnet 9 时，引入了 AppHostRelativeDotNet 机制，允许开发者自定义依赖框架发布的应用使用的 dotnet 运行时路径

<!--more-->


<!-- CreateTime:2025/04/26 07:13:38 -->

<!-- 发布 -->
<!-- 博客 -->

在 2022 时，我写了一个提案，允许应用程序自定义使用的 dotnet 运行时文件夹路径。详细请看 <https://github.com/dotnet/runtime/issues/64430>

这个提案的背景是我有很多个应用准备发布给到用户端上，如果这么多应用都走独立发布，自然会让用户的 C 盘充满重复的文件。如果是将 dotnet 运行时交给的是 Program File 文件夹下的全局文件夹，则可能会遇到各种被投毒问题，比如某次系统更新之后，应用程序就因为 .NET 环境损坏而无法启动

我所在的团队那会也在迁移一个大型的 .NET Framework 项目到 .NET 6 上，原本的项目会有多 exe 入口问题，这部分设计也改不动。多入口情况下也不适合每个入口都做独立发布，尽管独立发布的重复 BCL 等文件能够在安装包里面被压缩，但是在安装到用户设备上时，解压缩出来的内容依然会撑满用户的 C 盘

为此，我所在的团队就制作和开源了 <https://github.com/dotnet-campus/dotnetCampus.AppHost> 项目，细节原理请参阅 [如何让 .NET 程序脱离系统安装的 .NET 运行时独立运行？除了 Self-Contained 之外还有更好方法！谈 dotnetCampus.AppHost 的工作原理 - walterlv](https://blog.walterlv.com/post/how-does-the-dotnet-campus-apphost-work )

关于我所在的团队迁移大型项目的经验请参阅 [记将一个大型客户端应用项目迁移到 dotnet 6 的经验和决策](https://blog.lindexi.com/post/%E8%AE%B0%E5%B0%86%E4%B8%80%E4%B8%AA%E5%A4%A7%E5%9E%8B%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%94%E7%94%A8%E9%A1%B9%E7%9B%AE%E8%BF%81%E7%A7%BB%E5%88%B0-dotnet-6-%E7%9A%84%E7%BB%8F%E9%AA%8C%E5%92%8C%E5%86%B3%E7%AD%96.html )

我那会预期的情况是这样的，我在自己控制的路径下，如 `C:\Program Files\CompanyName` 文件夹下，放入了自己的 `DotNETRuntime[Version]` 文件夹。然后再依次部署上多个应用程序，这些应用程序都是采用依赖框架（Publish framework-dependent）方式发布，总的文件夹布局情况如下

```
C:\Program Files\CompanyName\DotNETRuntime[Version]\
C:\Program Files\CompanyName\Produce1\
C:\Program Files\CompanyName\Produce2\
C:\Program Files\CompanyName\Produce3\
```

如此即可让 Produce1 Produce2 Produce3 三个产品共用一个 dotnet 运行时

我的这个[提案](https://github.com/dotnet/runtime/issues/64430)被 dotnet 官方采纳了，加入到 .NET Host 提升计划里面，详细请看 <https://github.com/dotnet/runtime/issues/97931> 。经过了三年（实际上绝大部分时间都在讨论）的开发，终于在 dotnet 9 支持了这个功能，能够完全实现我预期的功能

此项功能被命名为 Embedded install location options for apphost ，被我翻译为嵌入 dotnet 安装路径到 AppHost 里的功能，我也对外宣称这是为依赖框架的应用自定义 .NET Runtime 文件夹路径的功能

接下来我将和大家介绍此功能的用法和效果

此功能涉及到的关键属性分别如下：

- AppHostDotNetSearch ： 决定从哪里开始寻找，可选参数为 AppLocal、 AppRelative、 EnvironmentVariables 和 Global，可认为在此之前就是 Global 的值。允许设置多个参数，多个参数之间依然用 `;` 分号隔开。在本文里面，核心功能将由 `AppRelative` 参数实现
- AppHostRelativeDotNet ： 配置相对于 exe 的路径，这个路径将被作为 dotnet 运行时的查找路径

默认情况下，可只需设置 AppHostRelativeDotNet 属性即可。当 AppHostRelativeDotNet 属性被设置的时候，隐式设置了 AppHostDotNetSearch 属性为 AppRelative 的值。但通常来讲，可以将 AppHostDotNetSearch 属性设置为 `AppHostDotNetSearch=AppRelative;Global` 的值，这就意味着如果从相对路径没有找到 dotnet 运行时，将自动回滚到从 Global 全局进行查找。这里的 Global 全局即 `C:\Program Files\dotnet\` 或 `C:\Program Files (x86)\dotnet\` 文件夹或全局注册表记录的路径

为了演示此功能的用法，我创建了一个名为 LinerewheldeholearjearHalllurlecayawfea 的控制台项目，控制台项目使用的是 .NET 9 默认控制台模版代码。编辑 csproj 项目文件，添加 AppHostDotNetSearch 和 AppHostRelativeDotNet 属性，修改之后的 csproj 项目文件代码大概如下。本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AppHostDotNetSearch>AppRelative;EnvironmentVariable;Global;</AppHostDotNetSearch>
    <AppHostRelativeDotNet>../relative/path/to/runtime</AppHostRelativeDotNet>
  </PropertyGroup>

</Project>
```

可以看到我在 AppHostRelativeDotNet 写入的是相对于 exe 的上一层文件夹空间的 `relative/path/to/runtime` 路径

准备工作就此完成，接下来就是设置进行框架依赖发布。这里需要特别说明的是 .NET Core （包含 .NET 5 和更高版本）的输出 exe 是不能实现 .NET Framework 的 AnyCpu 魔法的，在使用自定义 dotnet 运行时路径时，需要根据自己的需求，明确指定其版本。这里也需要额外说明的是，尽管本文内容都在 Windows 下测试，但事实上本文介绍的 dotnet 这项新功能是可以在全平台使用的，即在 Linux 或 mac 上也适用

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径0.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254252013475105.jpg)

我的发布配置文件 FolderProfile.pubxml 代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- https://go.microsoft.com/fwlink/?LinkID=208121. -->
<Project>
  <PropertyGroup>
    <Configuration>Release</Configuration>
    <Platform>Any CPU</Platform>
    <PublishDir>bin\Release\net9.0\publish\win-x86\</PublishDir>
    <PublishProtocol>FileSystem</PublishProtocol>
    <_TargetId>Folder</_TargetId>
    <TargetFramework>net9.0</TargetFramework>
    <RuntimeIdentifier>win-x86</RuntimeIdentifier>
    <SelfContained>false</SelfContained>
    <PublishSingleFile>false</PublishSingleFile>
    <PublishReadyToRun>false</PublishReadyToRun>
  </PropertyGroup>
</Project>
```

配置完成之后，直接进行发布，此时可以看到发布创建的文件只有几个。有了这项技术就不怕发布大量工具了，有了这项技术就可以让发布的 .NET Core（包含.NET 5及更高版本）应用也和 .NET Framework 应用一样小体积占用

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径1.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254252015201925.jpg)

发布完成之后，可不能和进行独立发布（Self-Contained）一样，直接就将此分发给到用户了，咱还需要对此进行包装文件夹布局

刚才在 AppHostRelativeDotNet 写的是相对于 `../relative/path/to/runtime` 文件夹，嗯，这里只能写相对文件夹路径，不能写绝对文件夹路径。那咱就需要将发布输出的文件包装为里一层文件夹，我这里选择将其放入到名为 App1 的文件夹里面，这样我如果有第二个应用，就可以放入到 App2 文件夹里面

再接着将 App1 文件夹放入到名为 App 的文件夹里面。再在 App 文件夹里面的 `relative/path/to/runtime` 文件夹里面放入 dotnet 运行时。如此就完成了包装文件夹布局，此时直接双击 `App\App1\LinerewheldeholearjearHalllurlecayawfea.exe` 就能运行了

包装完成的文件夹布局情况如下

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径2.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254252026203870.jpg)

```csharp
C:\LINDEXI\APP
|   
+---App1
|       LinerewheldeholearjearHalllurlecayawfea.deps.json
|       LinerewheldeholearjearHalllurlecayawfea.dll
|       LinerewheldeholearjearHalllurlecayawfea.exe
|       LinerewheldeholearjearHalllurlecayawfea.pdb
|       LinerewheldeholearjearHalllurlecayawfea.runtimeconfig.json
|       
\---relative
    \---path
        \---to
            \---runtime
                |   dotnet.exe
                |   LICENSE.txt
                |   ThirdPartyNotices.txt
                |   
                +---host
                |   \---fxr
                |       \---9.0.4
                |               hostfxr.dll
                |               
                \---shared
                    \---Microsoft.NETCore.App
                        \---9.0.4
                                .version
                                clretwrc.dll
                                clrgc.dll
                                clrjit.dll
                                coreclr.dll
                                createdump.exe
                                hostpolicy.dll
                                Microsoft.CSharp.dll
                                Microsoft.DiaSymReader.Native.x86.dll
                                Microsoft.NETCore.App.deps.json
                                Microsoft.NETCore.App.runtimeconfig.json
                                Microsoft.VisualBasic.Core.dll
                                Microsoft.VisualBasic.dll
                                Microsoft.Win32.Primitives.dll
                                Microsoft.Win32.Registry.dll
                                ...
                                System.Xml.XPath.dll
                                System.Xml.XPath.XDocument.dll
                                WindowsBase.dll

```

也许伙伴们有一个问题，那就是这里的 .NET Runtime 运行时文件夹组织是哪里来的，文件是从哪里来的。这是从 dotnet 官方下载的，下载链接是： <https://dotnet.microsoft.com/zh-cn/download/dotnet/9.0>

下载右边“运行应用 - 运行时”这一列的内容

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径3.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254252031363060.jpg)

运行时这一列有很多选项，具体应该下哪一个呢？这就看自己的需求了。如我只是一个简单的控制台，且准备发布的是 x86 应用，那我就应该下载 x86 二进制文件，就是这样的对应关系，先取决于要用什么框架，再决定用什么平台

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径4.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254252032403128.jpg)

下载下来的是一个 zip 压缩包，打开压缩包就可以看到这就是上文提到的 `relative/path/to/runtime` 文件夹内的结构，按照本文提供的方式将其解压缩就好了

<!-- ![](image/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径/dotnet 9 通过 AppHostRelativeDotNet 指定自定义的运行时路径5.png) -->
![](http://cdn.lindexi.site/lindexi%2F2025425203748178.jpg)

额外需要说明的是，现在对于桌面应用来说是没有提供二进制包，只有安装包。即对于 WPF 和 WinForms 来说，现在只有安装包可用。那咋办呢？很简单，只需要找一个干净的系统（如虚拟机内），下载安装包且安装。安装完成之后，即可在 `C:\Program Files\dotnet\` 或 `C:\Program Files (x86)\dotnet\` 文件夹内找到安装输出的文件，将其拷贝出来放入到 `relative/path/to/runtime` 文件夹内即可

通过此项技术，即可让多个应用共用一个私有分发的 .NET 运行时。也可以作为单应用多 exe 入口程序的共享运行时技术实现。这项技术对于小工具项目特别友好，避免小工具项目要么各自带着运行时独立发布，要么被第三方或系统投毒运行时的选择

既然这可以使用私有分发的 .NET 运行时，那对于一些动手能力强的开发者来说，也可以在这里面带上自己魔改之后的 .NET 版本，实现更多有趣的功能

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1e09f77a553d664872cb12324a118649ffd23ad9/Workbench/LinerewheldeholearjearHalllurlecayawfea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/1e09f77a553d664872cb12324a118649ffd23ad9/Workbench/LinerewheldeholearjearHalllurlecayawfea) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1e09f77a553d664872cb12324a118649ffd23ad9
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1e09f77a553d664872cb12324a118649ffd23ad9
```

获取代码之后，进入 Workbench/LinerewheldeholearjearHalllurlecayawfea 文件夹，即可获取到源代码

参考文档：

- https://github.com/dotnet/runtime/issues/97931
- https://github.com/dotnet/designs/blob/main/proposed/apphost-embed-install-location.md
- https://learn.microsoft.com/zh-cn/dotnet/core/project-sdk/msbuild-props
- https://github.com/dotnet/runtime/issues/64430
- https://github.com/dotnet/designs/blob/main/accepted/2020/install-locations.md

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

---

补充：

本文提供的方法对 .NET 9 框架有效。如期望能够在 TargetFramework 为 .NET 8 的低版本也采用此技术，可参阅以下方式

1. 先将 TargetFramework 调低到 .NET 8 版本
2. 寻找自己本地的 .NET 9 SDK 下的 apphost 文件，将其设置到 AppHostSourcePath 属性里

如我本地的 .NET 9 SDK 下的 x64 的 apphost 文件路径为：`C:\Program Files\dotnet\sdk\9.0.203\AppHostTemplate\apphost.exe`

我将其设置到 AppHostSourcePath 属性里面，其代码如下

```xml
    <AppHostSourcePath Condition="'$(RuntimeIdentifier)'=='win-x64'">C:\Program Files\dotnet\sdk\9.0.203\AppHostTemplate\apphost.exe</AppHostSourcePath>
```

如此依然按照上文提供的方法，也可采用相对路径引用运行时。此方法利用了 .NET 9 的 AppHost 兼容 .NET 8 版本的机制

不仅可以在控制台项目使用，也可以在带 -windows 的如 WPF 或 WinForms 项目上使用此技术。以下是我的 .NET 8 的 WPF 应用的 csproj 文件代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UseWpf>true</UseWpf>
    <AppHostDotNetSearch>AppRelative;Global;</AppHostDotNetSearch>
    <AppHostRelativeDotNet>../relative2</AppHostRelativeDotNet>
    <AppHostSourcePath Condition="'$(RuntimeIdentifier)'=='win-x64'">C:\Program Files\dotnet\sdk\9.0.203\AppHostTemplate\apphost.exe</AppHostSourcePath>
  </PropertyGroup>

</Project>
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b73d1145eef2d926fac050aeed1139424b60651f/Workbench/JeehaweejallheardallJefarrerlo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/b73d1145eef2d926fac050aeed1139424b60651f/Workbench/JeehaweejallheardallJefarrerlo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b73d1145eef2d926fac050aeed1139424b60651f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b73d1145eef2d926fac050aeed1139424b60651f
```

获取代码之后，进入 Workbench/JeehaweejallheardallJefarrerlo 文件夹，即可获取到源代码


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。