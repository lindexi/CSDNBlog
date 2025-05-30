
使用 BT 工具的时候，如果不小心开了超量的连接数，即在退出 BT 工具之后，依然存在很多连接记录，导致网速特别差的问题，即使在同个局域网进行传输，也能感受到网速特别差

<!--more-->


<!-- CreateTime:2025/04/08 07:06:09 -->

<!-- 发布 -->
<!-- 博客 -->

故事的背景是我在写一个 BT 客户端，不小心连接数写上天了，且里面小部分连接都建立上了。当我开始退出进程时，我就发现了网络速度是没有恢复的，即使和 PC 同局域网的手机在访问 dotnet serve 建立的 HTTP 服务，也很慢才能建立连接

本文提供的方法仅适合于现在 BT 工具已经退出了，如果 BT 工具都没退出，那可能就是网络资源全被 BT 吃掉了，和其他都没有关系

退出 BT 工具之后，无网速、或网络很慢才能建立连接，可尝试进行网络释放所有连接，从而断开 BT 现存的连接。这个过程里面，禁用网络适配器和插拔网线是无效的

打开管理员权限 cmd 终端，执行的命令如下

```
ipconfig /release
```

调用 `ipconfig /release` 命令之后会释放所有的连接，可能会见到短暂的网络断开现象

如果不放心的话，还可以继续配合 renew 命令更新所有适配器

```
ipconfig /renew
```

合起来的命令就是

```
ipconfig /release
ipconfig /renew
```

执行以上两句命令，如果网络依然没有恢复，还请看一下自己的路由器。看路由器是不是忙爆了，如果路由器过忙，且重启路由器过慢，可试试拔掉网线，让路由器清醒一下，如依然无效，那就重启路由器，断电重启也好


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。