
我就好奇无穷大和 NAN 在 C# 的二进制是如何表示的

<!--more-->


<!-- 发布 -->

揭开谜底

NAN: 00,00,00,00,00,00,F8,FF

正无穷 PositiveInfinity : 00,00,00,00,00,00,F0,7F

负无穷 NegativeInfinity : 00,00,00,00,00,00,F0,FF


这是小端的表示方法，实际上的值如下

NAN: 0xFFF8000000000000

正无穷 PositiveInfinity : 0x7FF0000000000000

负无穷 NegativeInfinity : 0xFFF0000000000000

测试方法如下

```csharp
        static void Main(string[] args)
        {
            MemoryStream stream = new MemoryStream();
            BinaryWriter writer = new BinaryWriter(stream);

            double db = double.NaN;
            writer.Write(db);

            Console.WriteLine(string.Join(",", stream.ToArray().Select(b => b.ToString("X2"))));

            stream.Seek(0, SeekOrigin.Begin);
            db = double.PositiveInfinity;
            writer.Write(db);
            Console.WriteLine(string.Join(",", stream.ToArray().Select(b => b.ToString("X2"))));

            stream.Seek(0, SeekOrigin.Begin);
            db = double.NegativeInfinity;
            writer.Write(db);
            Console.WriteLine(string.Join(",", stream.ToArray().Select(b => b.ToString("X2"))));
        }
```

这个值不是乱存的，有标准，请看 [IEEE 754 - Wikipedia](https://en.wikipedia.org/wiki/IEEE_754 )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。