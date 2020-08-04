
假定我有一个文件的路径，我将这个文件路径放在文件夹判断方法里面，请问此时返回的是存在还是不存在？答案是返回不存在

<!--more-->


<!-- CreateTime:2020/8/1 10:17:31 -->

<!-- 发布 -->

如下面测试代码，这里的 GelteajoutrerebaKoutigasremawcho.dll 就是程序集，也就是文件是存在的，那么放在 DirectoryInfo 判断输出的内容是否存在

```csharp
namespace GelteajoutrerebaKoutigasremawcho
{
    class Program
    {
        static void Main(string[] args)
        {
            DirectoryInfo dir = new DirectoryInfo(@"GelteajoutrerebaKoutigasremawcho.dll");
            Console.WriteLine(dir.Exists);
            
            var file = new FileInfo(@"GelteajoutrerebaKoutigasremawcho.dll");
            Console.WriteLine(file.Exists);
        }
    }
}
```

输出是 False True 也就是判断文件夹不存在，判断文件存在

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e237082b643c86cd15124f201c82f46955b9ab84/GelteajoutrerebaKoutigasremawcho) 欢迎小伙伴访问



我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。