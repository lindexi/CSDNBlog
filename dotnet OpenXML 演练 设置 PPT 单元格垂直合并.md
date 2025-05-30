
本文将告诉大家如何在使用 2.20 版本的 OpenXML SDK 对 PPT 里面的单元格进行垂直合并

<!--more-->


<!-- CreateTime:2025/04/15 07:19:14 -->
<!-- 发布 -->
<!-- 博客 -->

一般在做 OpenXML 相关生成或处理任务的时候，都会预先做好模版的文件，如本文这里预先做好了 Test.pptx 文档

<!-- ![](image/dotnet OpenXML 演练 设置 PPT 单元格垂直合并/dotnet OpenXML 演练 设置 PPT 单元格垂直合并0.png) -->
![](http://cdn.lindexi.site/lindexi%2F20254161938171020.jpg)

这份文档里面只包含了一个 a1b2 的表格，本文演练的需求就是将 a 和 1 所在的单元格进行垂直合并

在 PPT 里面合并单元格时，重点就是在首个单元格设置好 RowSpan 属性，这个属性用于设置从这个单元格以下多少个单元格将被合并，数值包含自己在内，即最小合并单元为 2 个单元格。其次是非首个单元格的，需要设置 VerticalMerge 属性

按照这份文档，可以看到很简单的先取出首行首列，和末行首列，这就是 a 和 1 所在的单元格，代码如下

```csharp
Table table = ...
var firstRow = table.Elements<TableRow>().First();
var lastRow = table.Elements<TableRow>().Last();

var firstRowFirstCell = firstRow.Elements<TableCell>().First();
var lastRowFirstCell = lastRow.Elements<TableCell>().First();
```

对首个单元格设置上 RowSpan 属性，这里只合并两个单元格，设置 2 的值。对末行首列单元格的 VerticalMerge 设置 true 属性，代码如下

```csharp
firstRowFirstCell.RowSpan = 2;
lastRowFirstCell.VerticalMerge = true;
```

完全的代码如下

```csharp
using System;
using System.Linq;

using DocumentFormat.OpenXml.Drawing;
using DocumentFormat.OpenXml.Presentation;

using GraphicFrame = DocumentFormat.OpenXml.Presentation.GraphicFrame;
using NonVisualDrawingProperties = DocumentFormat.OpenXml.Presentation.NonVisualDrawingProperties;
using NonVisualShapeProperties = DocumentFormat.OpenXml.Presentation.NonVisualShapeProperties;

using var presentationDocument =
    DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("Test.pptx", true);
var presentationPart = presentationDocument.PresentationPart;
var slidePart = presentationPart!.SlideParts.First();
var slide = slidePart.Slide;

var graphicFrame = slide.CommonSlideData!.ShapeTree!.GetFirstChild<GraphicFrame>()!;
var graphic = graphicFrame.Graphic!;
var graphicData = graphic.GraphicData!;
var table = graphicData.GetFirstChild<Table>()!; // a:tbl
var firstRow = table.Elements<TableRow>().First();
var lastRow = table.Elements<TableRow>().Last();

var firstRowFirstCell = firstRow.Elements<TableCell>().First();
var lastRowFirstCell = lastRow.Elements<TableCell>().First();

firstRowFirstCell.RowSpan = 2;
lastRowFirstCell.VerticalMerge = true;
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0/Pptx/GayurjabeaNiyegacher) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0/Pptx/GayurjabeaNiyegacher) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0
```

获取代码之后，进入 Pptx/GayurjabeaNiyegacher 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。