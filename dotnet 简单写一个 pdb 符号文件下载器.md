
本文将告诉大家如何读取 PE 文件头信息，拼接 PDB 符号文件下载地址，从微软公共符号服务器拉取符号文件

<!--more-->


<!-- CreateTime:2024/11/27 07:23:23 -->

<!-- 发布 -->
<!-- 博客 -->

本文将以拉取 ntdll.dll 为例子告诉大家如何从 msdl.microsoft.com 下载符号

我先将自己电脑上的 ntdll.dll 拷贝到输出路径，方便我进行访问。读取 C 盘的 Windows 文件夹的 DLL 文件也是有读取权限的，即不拷贝到输出路径也不会有任何的问题的

整体步骤是先读取 dll 的 PE 文件信息，获取到调试信息，即 PDB 名加 Guid 和 Age 参数，由以上三个参数拼接为下载地址

本文使用的 PEReader 等辅助类为 .NET Core 1.0 引入，如需在 .NET Framework 使用，请安装 [System.Reflection.Metadata](https://www.nuget.org/packages/System.Reflection.Metadata ) 库

本文建立的是 dotnet 9 的控制台文件

先使用 File.OpenRead 打开要读取的 DLL 文件，代码如下

```csharp
var file = @"ntdll.dll";

using var fileStream = File.OpenRead(file);
```

接着放入到 PEReader 里面读取，如此即可非常方便完成对 PE 文件的读取，代码如下

```csharp
var peReader = new PEReader(fileStream);
```

尝试读取传入的 PE 文件的调试目录信息，代码如下

```csharp
var debugDirectoryEntries = peReader.ReadDebugDirectory();
```

这里的调试目录可能有很多，咱只需使用 DebugDirectoryEntryType.CodeView 类型的即可

```csharp
foreach (var debugDirectoryEntry in debugDirectoryEntries)
{
    if (debugDirectoryEntry.Type != DebugDirectoryEntryType.CodeView)
    {
        continue;
    }

    ...
}
```

在 DebugDirectoryEntry 里面存放的只是一些地址偏移和大小的信息，需要进一步调用 ReadCodeViewDebugDirectoryData 方法读取到更多信息，代码如下

```csharp
foreach (var debugDirectoryEntry in debugDirectoryEntries)
{
    if (debugDirectoryEntry.Type != DebugDirectoryEntryType.CodeView)
    {
        continue;
    }

    var readCodeViewDebugDirectoryData = peReader.ReadCodeViewDebugDirectoryData(debugDirectoryEntry);
    var path = readCodeViewDebugDirectoryData.Path;
    var guid = readCodeViewDebugDirectoryData.Guid;
    var age = readCodeViewDebugDirectoryData.Age;

    var pdbName = path;

    ...
}
```

如此即可获取到下载地址拼接的信息

```csharp
    var downloadUrl = $"http://msdl.microsoft.com/download/symbols/{pdbName}/{(guid.ToString("N").ToUpperInvariant() + age.ToString())}/{pdbName}";
```

继续使用 HttpClient 将其下载下来，代码如下

```csharp
    var httpClient = new HttpClient();

    using var httpResponseMessage = await httpClient.GetAsync(downloadUrl, HttpCompletionOption.ResponseHeadersRead);

    var pdbFile = Path.GetFullPath(pdbName);
    await using var downloadFileStream = File.Create(pdbFile);
    await httpResponseMessage.Content.CopyToAsync(downloadFileStream);
```

下载文件的方法十分简单，先是准备要下载的文件存放的路径，我这里使用的是相对工作路径的方式，即 `var pdbFile = Path.GetFullPath(pdbName);` 简单的方式拿到绝对路径。这里获取绝对路径仅仅只是为了方便调试而已，无实际逻辑意义

接着使用 File.Create 方法创建文件，且返回 FileStream 对象，方便进行下载内容的 CopyToAsync 写入到文件

以上代码的另一个细节是请求的时候带上了 HttpCompletionOption.ResponseHeadersRead 参数，带上这个参数将告诉 HttpClient 不要等待内容都拉取完了再让 GetAsync 方法返回，而是只要 http 的头信息拿到了，就应该返回了。这一点对下载文件来说，比较有优化，大部分下载的文件的文件长度都不小，全等待下载完成再让 GetAsync 返回，再拷贝到文件，这个逻辑相对来说是比较亏的。只有对 API 调用形的后台访问，才合适使用默认的 HttpCompletionOption.ResponseContentRead 参数，全部完成再返回，如此可以更简化异常处理情况，确保网络通讯完成再返回

通过上文简单的方式即可完成对 DLL 的符号文件下载

以上代码其实还隐藏了另一个功能，那就是自己组建符号服务器，可以自己在构建完成之后，根据如上信息，将 PDB 符号文件存放到合适的路径里面或记录到数据库里面，不依赖 symstore 工具

本文的 Program.cs 代码如下

```csharp
using System.Reflection.PortableExecutable;
using System.Runtime.InteropServices;
using System.Text;

var file = @"ntdll.dll";

using var fileStream = File.OpenRead(file);

var peReader = new PEReader(fileStream);
var httpClient = new HttpClient();

var debugDirectoryEntries = peReader.ReadDebugDirectory();
foreach (var debugDirectoryEntry in debugDirectoryEntries)
{
    if (debugDirectoryEntry.Type != DebugDirectoryEntryType.CodeView)
    {
        continue;
    }

    var readCodeViewDebugDirectoryData = peReader.ReadCodeViewDebugDirectoryData(debugDirectoryEntry);
    var path = readCodeViewDebugDirectoryData.Path;
    var guid = readCodeViewDebugDirectoryData.Guid;
    var age = readCodeViewDebugDirectoryData.Age;

    var pdbName = path;

    var downloadUrl = $"http://msdl.microsoft.com/download/symbols/{pdbName}/{(guid.ToString("N").ToUpperInvariant() + age.ToString())}/{pdbName}";
    using var httpResponseMessage = await httpClient.GetAsync(downloadUrl, HttpCompletionOption.ResponseHeadersRead);

    var pdbFile = Path.GetFullPath(pdbName);
    await using var downloadFileStream = File.Create(pdbFile);
    await httpResponseMessage.Content.CopyToAsync(downloadFileStream);
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6bc0a171f53f4da8a968257c621c9df7a6de77a3/Workbench/WerjiwaijogoNemcuryecho) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6bc0a171f53f4da8a968257c621c9df7a6de77a3/Workbench/WerjiwaijogoNemcuryecho) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6bc0a171f53f4da8a968257c621c9df7a6de77a3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6bc0a171f53f4da8a968257c621c9df7a6de77a3
```

获取代码之后，进入 Workbench/WerjiwaijogoNemcuryecho 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。