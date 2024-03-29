
本文记录一个 WPF 的已知问题，在通过 CollectionViewSource 获取到 CollectionView 之后，如果 CollectionViewSource 对象已被 GC 回收，将可能在调用 CollectionView 的 SortDescriptions 属性进行清空或者移除项时，也就是使用 SortDescriptionCollection 类型的清空或者移除项时，在 WPF 框架里面抛出空异常

<!--more-->


<!-- CreateTime:2022/12/29 8:43:24 -->


<!-- 发布 -->
<!-- 博客 -->

此问题已经报告给 WPF 官方，请看 [https://github.com/dotnet/wpf/issues/7389](https://github.com/dotnet/wpf/issues/7389 )

我现在是一个成熟的开发者了，自己报告的 BUG 就要自己修。此问题已修复，请看 [https://github.com/dotnet/wpf/pull/7390](https://github.com/dotnet/wpf/pull/7390)

此问题的复现步骤如下

在一个 WPF 项目里面，构建出一个 CollectionViewSource 对象，接着只获取存放此 CollectionViewSource 对象的 View 属性，此 View 属性就是 CollectionView 类型的一个对象，将 CollectionView 存放到字段里面。等待 CollectionViewSource 被回收之后，调用 CollectionView 的 SortDescriptions 属性进行清空 SortDescriptionCollection 的内容。代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        var collectionViewSource = new CollectionViewSource()
        {
            Source = new List<Foo>(),
            IsLiveSortingRequested = true,
        };

        var collectionView = collectionViewSource.View;
        _collectionView = collectionView;

        collectionView.SortDescriptions.Add(new SortDescription("Name", ListSortDirection.Descending));

        Loaded += MainWindow_Loaded;
    }

    private readonly ICollectionView _collectionView;

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        GC.Collect();
        GC.WaitForFullGCComplete();
        GC.Collect();
    }

    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        _collectionView.SortDescriptions.Clear();
    }
```

以上的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c7556d7b92605000011425f82793f9e4063e5a00/LechelaneHenayfucee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c7556d7b92605000011425f82793f9e4063e5a00/LechelaneHenayfucee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c7556d7b92605000011425f82793f9e4063e5a00
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c7556d7b92605000011425f82793f9e4063e5a00
```

获取代码之后，进入 LechelaneHenayfucee 文件夹

运行代码，然后点击按钮，就可以看到在 WPF 框架里面抛出空异常


异常的调用堆栈大概如下


```
>	PresentationFramework.dll!System.Windows.Data.ListCollectionView.PrepareLocalArray() 
 	PresentationFramework.dll!System.Windows.Data.ListCollectionView.RefreshOverride() 
 	PresentationFramework.dll!System.Windows.Data.CollectionView.RefreshInternal() 
 	PresentationFramework.dll!System.Windows.Data.CollectionView.RefreshOrDefer() 
 	PresentationFramework.dll!System.Windows.Data.ListCollectionView.SortDescriptionsChanged(object sender, System.Collections.Specialized.NotifyCollectionChangedEventArgs e) 
 	WindowsBase.dll!System.ComponentModel.SortDescriptionCollection.RemoveItem(int index)
 	System.Private.CoreLib.dll!System.Collections.ObjectModel.Collection<System.ComponentModel.SortDescription>.RemoveAt(int index)
 	App.dll!MyClass.Foo();
```

阅读 WPF 框架的源代码，可以了解到原因就是因为 CollectionViewSource 对象没有被引用，从而被 GC 回收。在 CollectionViewSource 回收之后，将会让其 View 属性，也就是 CollectionView 类型，被 WPF 框架触发 DetachFromSourceCollection 方法进行回收。这个 DetachFromSourceCollection 方法代码如下


```csharp
 public virtual void DetachFromSourceCollection() 
 { 
     INotifyCollectionChanged incc = _sourceCollection as INotifyCollectionChanged; 
     if (incc != null) 
     { 
         IBindingList ibl; 
         if (!(this is BindingListCollectionView) || 
             ((ibl = _sourceCollection as IBindingList) != null && !ibl.SupportsChangeNotification)) 
         { 
             incc.CollectionChanged -= new NotifyCollectionChangedEventHandler(OnCollectionChanged); 
         } 
     } 
  
     _sourceCollection = null; 
 }
```

在 DetachFromSourceCollection 方法里面，将 `_sourceCollection` 设置为空，这就导致了在清空 SortDescriptionCollection 内容的时候，尝试获取 `_sourceCollection` 的属性时，抛出空异常


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。