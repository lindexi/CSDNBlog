
本文告诉大家如何修复使用 GitLab 的 Runner 做 CI 时提示 "pwsh": executable file not found in %PATH% 错误

<!--more-->


<!-- CreateTime:2023/3/2 20:13:31 -->

<!-- 发布 -->


有两个方法，第一个方法就是安装 pwsh 命令，安装方法是在 PowerShell 里输入以下代码安装

```
winget install Microsoft.PowerShell
```

如果嫌弃 winget 输入太慢，可以从他的输出里面找到 PowerShell 的下载地址，换个快速的下载器去下载即可

输入以上命令之后，相信你看界面就会了

第二个方法就是将 `pwsh` 修改为 `powershell` 代码，编辑 gitlab-runner.exe 所在文件夹下的 config.toml 文件，将里面的  `pwsh` 修改为 `powershell` 如以下代码

```
  name = "xxxxx"
  url = "https://xxxxxx/"
  id = 1363
  token = "h_h-xxx-"
  token_obtained_at = 2023-03-02T11:00:05Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "shell"
  shell = "powershell" // 只需要关键这句话就可以了。原本是 `shell = "pwsh"` 这句话
```

两个方法选一个即可，我比较推荐使用第一个方法


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。