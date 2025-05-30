
本文将告诉大家如何在 C# dotnet 里面，调用 Windows 的 EnumDisplayDevices 函数获取显示设备信息

<!--more-->


<!-- CreateTime:2025/05/01 07:07:49 -->

<!-- 发布 -->
<!-- 博客 -->

核心代码如下

```csharp
DISPLAY_DEVICEW displayDevice = default;
displayDevice.cb = (uint) Marshal.SizeOf(typeof(DISPLAY_DEVICEW));

for (uint id = 0; EnumDisplayDevices(null, id, ref displayDevice, 0); id++)
{
    var deviceName = displayDevice.DeviceName.ToString();
    var deviceString = displayDevice.DeviceString.ToString();
    var deviceKey = displayDevice.DeviceKey.ToString();
    var deviceId = displayDevice.DeviceID.ToString();

    Console.WriteLine($"EnumDisplayDevices");
    Console.WriteLine($"DeviceName={deviceName}");
    Console.WriteLine($"DeviceString={deviceString}");
    Console.WriteLine($"DeviceKey={deviceKey}");
    Console.WriteLine($"DeviceID={deviceId}");
    Console.WriteLine();
}
```

以上代码缺少的 Win32 方法定义和结构体定义是通过 CsWin32 库自动生成的，详细请看 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

运行以上代码的控制台输出信息如下

```
EnumDisplayDevices
DeviceName=\\.\DISPLAY1
DeviceString=Intel(R) UHD Graphics
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{5F41D0AE-D6C9-11EF-98F4-E0364B98FD61}\0000
DeviceID=PCI\VEN_8086&DEV_A780&SUBSYS_88821043&REV_04

EnumDisplayDevices
DeviceName=\\.\DISPLAY2
DeviceString=Intel(R) UHD Graphics
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{5F41D0AE-D6C9-11EF-98F4-E0364B98FD61}\0001
DeviceID=PCI\VEN_8086&DEV_A780&SUBSYS_88821043&REV_04

EnumDisplayDevices
DeviceName=\\.\DISPLAY3
DeviceString=Intel(R) UHD Graphics
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{5F41D0AE-D6C9-11EF-98F4-E0364B98FD61}\0002
DeviceID=PCI\VEN_8086&DEV_A780&SUBSYS_88821043&REV_04

EnumDisplayDevices
DeviceName=\\.\DISPLAY4
DeviceString=Intel(R) UHD Graphics
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{5F41D0AE-D6C9-11EF-98F4-E0364B98FD61}\0003
DeviceID=PCI\VEN_8086&DEV_A780&SUBSYS_88821043&REV_04

EnumDisplayDevices
DeviceName=\\.\DISPLAY5
DeviceString=OrayIddDriver Device
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{0FEA508F-0EAA-11F0-98F8-005056C00008}\0000
DeviceID=Root\OrayIddDriver

EnumDisplayDevices
DeviceName=\\.\DISPLAY6
DeviceString=OrayIddDriver Device
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{0FEA508F-0EAA-11F0-98F8-005056C00008}\0001
DeviceID=Root\OrayIddDriver

EnumDisplayDevices
DeviceName=\\.\DISPLAY7
DeviceString=OrayIddDriver Device
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{0FEA508F-0EAA-11F0-98F8-005056C00008}\0002
DeviceID=Root\OrayIddDriver

EnumDisplayDevices
DeviceName=\\.\DISPLAY8
DeviceString=OrayIddDriver Device
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{0FEA508F-0EAA-11F0-98F8-005056C00008}\0003
DeviceID=Root\OrayIddDriver

EnumDisplayDevices
DeviceName=\\.\DISPLAY9
DeviceString=OrayIddDriver Device
DeviceKey=\Registry\Machine\System\CurrentControlSet\Control\Video\{0FEA508F-0EAA-11F0-98F8-005056C00008}\0004
DeviceID=Root\OrayIddDriver
```

我装了向日葵了，于是显示设备会看起来更多了

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d1a7014b263d956cc15edd78b8a428d264338c4f/Workbench/CelgajayweWhilalelheyar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/d1a7014b263d956cc15edd78b8a428d264338c4f/Workbench/CelgajayweWhilalelheyar) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d1a7014b263d956cc15edd78b8a428d264338c4f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin d1a7014b263d956cc15edd78b8a428d264338c4f
```

获取代码之后，进入 Workbench/CelgajayweWhilalelheyar 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。