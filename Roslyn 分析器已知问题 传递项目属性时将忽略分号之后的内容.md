
本文记录 Roslyn 分析器、源代码生成器的已知问题，通过CompilerVisibleProperty 传递值时，所有在 `;`、`#` 和换行符之后的字符都会被忽略

<!--more-->


<!-- CreateTime:2025/03/12 07:07:45 -->

<!-- 发布 -->
<!-- 博客 -->

相关问题链接：

- <https://github.com/dotnet/roslyn/issues/43970>
- <https://github.com/dotnet/roslyn/issues/51692>

此问题由 [walterlv](https://github.com/walterlv) 发现，我只是一个记录问题的工具人

## 解决方案

将需要传递的值进行 Base64 编码，避免 `;`、`#` 和换行符的影响。

此方案来自于： <https://github.com/dotnet/roslyn/issues/51692>

不足之处，这是一个十分奇怪的行为，先 Base64 编码，再 Base64 解码。这个过程只是为了断开表达式关系，代码也不优雅

### 其他无效的解决方案

1.是否可以通过 `Replace` 来替换掉 `;` 字符？

经过测试，此方案仅在原属性是字符串常量时才正常工作；当原属性是 MSBuild 计算出来的值时，`Replace` 会导致把计算过程表达式当作字符串常量来处理，导致替换完之后表达式变成了字符串结果了。

2.是否可在 Target 中替换字符？

经过测试，在多个 Target 中替换都会得到跟上述相同的结果。

3.是否可通过 `$([MSBuild]::Escape('$(Foo)'))` 转义来规避？

经过测试，会得到跟上述相同的结果

## 文档

微软官方的 CookBook 中完全没有提及此问题：

- <https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.cookbook.md>

有网友主动为 CookBook 提交了此问题的规避方案： <https://github.com/dotnet/roslyn/pull/76818>

不过此规避方案也仅在字符串常量时有效，不适用于本方案


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。