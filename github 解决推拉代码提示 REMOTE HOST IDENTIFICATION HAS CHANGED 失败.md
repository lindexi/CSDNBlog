
本文记录最近 github 推送或拉取代码时提示 WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! 而失败的解决方法

<!--more-->


<!-- CreateTime:2023/3/27 8:39:35 -->


<!-- 发布 -->
<!-- 博客 -->

报错提示如下

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s.
Please contact your system administrator.
Add correct host key in /c/Users/lindexi/.ssh/known_hosts to get rid of this message.
Offending RSA key in /c/Users/lindexi/.ssh/known_hosts:2
Host key for github.com has changed and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

最简单的方式就是删除上面提到的 `known_hosts` 文件，例如我的文件是 `/c/Users/lindexi/.ssh/known_hosts` 路径，直接使用 rm 命令删除，代码如下

```
rm /c/Users/lindexi/.ssh/known_hosts
```

还请在你电脑上执行以上命令的时候，替换为你自己的电脑上的路径

执行完成之后，重新推拉代码，可以看到以下提示信息，只需要输入 yes 回车即可

```
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

以上步骤就是在将 github 添加到 `known_hosts` 文件里

也就是下一次推拉代码就不需要再次执行以上步骤

为什么有这样的事情呢？这是因为在 2023.03.24 更换了 GitHub 的 RSA 的 Key 值。因为 GitHub 团队发现自己的 Key 泄露了。详细请看 [We updated our RSA SSH host key The GitHub Blog](https://github.blog/2023-03-23-we-updated-our-rsa-ssh-host-key/ )


我搭建了自己的博客 [https://blog.lindexi.com/](https://blog.lindexi.com/) 欢迎大家访问，里面有很多新的博客。只有在我看到博客写成熟之后才会放在csdn或博客园，但是一旦发布了就不再更新

如果在博客看到有任何不懂的，欢迎交流，我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入

如有不方便在博客评论的问题，可以加我 QQ 2844808902 交流

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。