
本文记录 SQLite.Interop.dll 由于 mscoree.dll 文件损坏而在加载时抛出 BadImageFormatException 错误，错误信息是 0x8007000B 试图加载格式不正确的程序

<!--more-->


<!-- CreateTime:2024/09/05 07:21:42 -->

<!-- 发布 -->
<!-- 博客 -->

表现行为是在开发者电脑上能好好运行，在大部分用户设备上能好好运行，只有在个别用户的设备上运行报错

对应的英文错误信息如下

```
An attempt was made to load a program with an incorrect format. (Exception from HRESULT: 0x8007000B)
```

常见的错误堆栈如下

```
System.BadImageFormatException: 试图加载格式不正确的程序。 (0x8007000B)
   in SQLiteErrorCode UnsafeNativeMethods.sqlite3_open_interop(byte[] utf8Filename, byte[] vfsName, SQLiteOpenFlagsEnum flags, int extFuncs, ref IntPtr db)
   in void SQLite3.Open(string strFilename, string vfsName, SQLiteConnectionFlags connectionFlags, SQLiteOpenFlagsEnum openFlags, int maxPoolSize, bool usePool)
   in void SQLiteConnection.Open()
   in void AsyncDbConnection.Open()
   in IAsyncDbConnection DataConnection.EnsureConnection(bool connect)
   in DbConnection DataConnection.get_Connection()
   in int DataConnection.LinqToDB.Common.Internal.IConfigurationID.get_ConfigurationID()
   in new Query(IDataContext dataContext, Expression expression)
   in new Query<T>(IDataContext dataContext, Expression expression)
   in Query<T> Query<T>.CreateQuery(ExpressionTreeOptimizationContext optimizationContext, ParametersContext parametersContext, IDataContext dataContext, Expression expr)
   in Query<T> Query<T>.GetQuery(IDataContext dataContext, ref Expression expr, out bool dependsOnParameters)
   in Query<T> ExpressionQuery<T>.GetQuery(ref Expression expression, bool cache, out bool dependsOnParameters)
   in TResult ExpressionQuery<T>.System.Linq.IQueryProvider.Execute<TResult>(Expression expression)
   in TSource Queryable.FirstOrDefault<TSource>(IQueryable<TSource> source, Expression<Func<TSource, bool>> predicate)
   in LocalCoursewareEntry CoursewareSqliteDbReader.GetCoursewareEntry(string localId)+(ITable<LocalCoursewareEntry> tb) => { }
   in TReturn CoursewareSqliteDbOperator.ExecuteOnCoursewareTable<TReturn>(Func<ITable<LocalCoursewareEntry>, TReturn> query)+(DataConnection conn) => { }
   in TReturn SqliteDbOperator.ExecuteQuery<TReturn>(Func<DataConnection, TReturn> func)
```

或

```
System.BadImageFormatException: 试图加载格式不正确的程序。 (0x8007000B)
  ?, in SQLiteErrorCode UnsafeNativeMethods.sqlite3_config_none(SQLiteConfigOpsEnum op)
  ?, in bool SQLite3.StaticIsInitialized()
```

错误原因是 mscoree.dll 文件损坏，这个 mscoree.dll 不是放在应用程序文件夹下的，所在的系统路径如下

- x64: `C:\Windows\System32\mscoree.dll`
- x86: `C:\Windows\SysWOW64\mscoree.dll`

如果当前能抓到复现的系统环境，可以尝试用自己开发机正常的 mscoree.dll 替换出现问题的机器的文件，预期替换之后能正常工作。如替换之后不能正常工作，可能是错误将 x86 和 x64 文件给混了，或者是 SQLite.Interop.dll 文件本身或其他依赖出错

此问题如使用 [Dependencies](https://github.com/lucasg/Dependencies) 工具，将 SQLite.Interop.dll 文件拖入，则可以看到 mscoree.dll 被打了感叹号

我尝试将一个用户设备上的 mscoree.dll 拖回来分析，使用 `dumpbin` 工具分析提示无效的格式文件

```
C:\Program Files\Microsoft Visual Studio\2022\Professional>dumpbin C:\lindexi\Work\mscoree.dll
Microsoft (R) COFF/PE Dumper Version 14.40.33811.0
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file C:\lindexi\Work\mscoree.dll
C:\lindexi\Work\mscoree.dll : warning LNK4048: 无效的格式文件；已忽略

  Summary
```

尝试使用二进制查看工具打开 mscoree.dll 文件

读取到的内容完全不是 PE 文件格式，证明文件完全被损坏

我没有继续了解为什么 mscoree.dll 文件出错了


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。