
我将在一个 .NET Framework 项目升级到 dotnet 6 时发现构建不通过，因为原先的代码使用到了 EnumeratorToEnumVariantMarshaler 类型，在 dotnet 6 里面找不到。本文将告诉大家如何修复此问题

<!--more-->


<!-- CreateTime:2023/3/28 14:54:11 -->


<!-- 发布 -->
<!-- 博客 -->

在 .NET Framework 定义的 [EnumeratorToEnumVariantMarshaler](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.custommarshalers.enumeratortoenumvariantmarshaler ) 类型是在 dotnet core 里缺失的

代替方法是使用 MarshalType 属性写字符串，如以下代码

```csharp
        [MethodImpl(MethodImplOptions.InternalCall)]
        [DispId(1)]
        [TypeLibFunc(1)]
        [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalType = "System.Runtime.InteropServices.CustomMarshalers.EnumeratorToEnumVariantMarshaler")]
        IEnumerator GetEnumerator();
```


参考文档

[CustomMarshalers - EnumeratorToEnumVariantMarshaler · Issue #47243 · dotnet/runtime](https://github.com/dotnet/runtime/issues/47243 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。