
如果使用某个变量去获取某个类型的对象创建，但是在这个类型的构造函数调用时抛出异常，请问此变量是否可以拿到对应的对象

<!--more-->


<!-- CreateTime:2021/6/17 20:14:09 -->

<!-- 发布 -->

如下面代码

```csharp
        private void F1()
        {
            Foo foo = null;
            try
            {
                foo = new Foo();
            }
            catch
            {
                // 忽略
            }
        }

    class Foo
    {
        public Foo()
        {
            throw new Exception("lindexi is doubi");
        }

        ~Foo()
        {
        }
    }
```

请问在执行完成 F1 函数前，在 F1 函数定义的 foo 变量是什么，是空，还是 Foo 对象

答案自然是空，原因是在 .NET 运行时的逻辑是先分配对象内存空间，然后再调用对象的构造函数，接着将对象赋值给到 foo 变量

而在进行第二步时就炸了，自然就不会给 foo 变量赋值




我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。