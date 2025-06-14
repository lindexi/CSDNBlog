
本文记录一个 ReSharper 已知问题，在在 Directory.Build.props 属性求值结果不能应用到项目里，此行为将导致 ReSharper 获取的构建文件与 VisualStudio 不同，导致了可能存在的冲突

<!--more-->


<!-- CreateTime:2025/06/10 07:01:43 -->

<!-- 发布 -->
<!-- 博客 -->

此问题我已经报告给 ReSharper 官方，详细请看： <https://youtrack.jetbrains.com/issue/RSRP-500675>

以下是我的报告内容：

ReSharper Reports 'Ambiguous reference' Error Incorrectly in Project with Cross - Project References

If a project simultaneously marks and references the code of another project or conducts project references, an "Ambiguous reference" error may appear in ReSharper. However, the project can actually be built successfully in Visual Studio.

The project structure to reproduce this issue is a bit complex, but don't worry. I've built it. I've placed the minimal reproducible code on GitHub, and you can find all the code via this link https://github.com/lindexi/lindexi_gd/tree/40067cf83fece460a7d80c3d2445e6ab652531b3/Workbench/QeedekallheweWhanenurnellere .
Here are the steps to reproduce:

1. Create three new projects: Lib, AllInOne, and App projects.
2. Put the Directory.Build.props file in the folder where the sln file is located.
3. Establish the association between projects: The AllInOne project depends on the Lib project, and the App project depends on the AllInOne project.

After creating the projects, fill in the necessary content.

Add the following custom attribute content to the Directory.Build.props file:

```xml
  <PropertyGroup>
    <PackAllInOne>true</PackAllInOne>
    <PackAllInOne Condition="'$(Configuration)'=='debug'">false</PackAllInOne>
  </PropertyGroup>
```

Add an empty Foo type to the Lib project with no code inside, as only the type is needed to reproduce this issue.

```csharp
namespace Lib;

public class Foo
{
}
```

Write the Program.cs file in the App project to create a Foo object. The code is as follows:
csharp

```csharp
using Lib;

var foo = new Foo();
```

Up to this point, everything works fine. This is because the App project indirectly depends on the Lib project through the AllInOne project and can access the Foo type.

The final step is to edit the content of the AllInOne's csproj file. Configure the AllInOne project to decide whether to reference the Lib project or the code inside the Lib project through the PackAllInOne property. The code is as follows:

```xml
  <ItemGroup Condition=" $(PackAllInOne) == true ">
    <Compile Include="..\Lib\**\*.cs" />
  </ItemGroup>

  <ItemGroup Condition=" $(PackAllInOne) != true ">
    <ProjectReference Include="..\Lib\Lib.csproj" />
  </ItemGroup>
```

After completing this step, you'll see that the `Program.cs` file, which originally worked fine in the App project, now shows an "Ambiguous reference" error in ReSharper. Even though the project can still be built successfully in Visual Studio, it means ReSharper's error message is an incorrect false alarm.

You can find all my code in https://github.com/lindexi/lindexi_gd/tree/40067cf83fece460a7d80c3d2445e6ab652531b3/Workbench/QeedekallheweWhanenurnellere


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。