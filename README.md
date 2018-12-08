由于我比较笨还英文严重不好，所以按照 [Gitalk](https://github.com/gitalk/gitalk) 提供的文档给 Hexo 添加评论功能，简直就是不能行！各种报错后，翻看 Issues、Google、百度... 用了一天时间终于 Gitalk 可用了。
这里我分享一下我的添加过程与报错问题的解决方法，供参考借鉴！
<!--more-->

## 添加过程

### 申请 GitHub Application

如果有的话在 Github 中 `settings / Developer settings` 选择一个 OAuth App，如果没有 [点击这里申请](https://github.com/settings/applications/new)，我是第一次使用，自然需要从申请开始。

![注册需要填写的表单](https://github.com/zhwangart/gitalk/raw/master/source/Register-OAuth-application.png)

说明一下：

+ Application name: 没有限制的 APP 名称，比如：Hexo-application
+ Homepage URL: 网站的主页，我理解就是根目录，比如：[https://zhwangart.github.io](https://zhwangart.github.io)
+ Application description: 描述，非必填，我当时没有写。
+ Authorization callback URL: 回调 url 我理解就是网站的域名，比如：https://zhwangart.github.io

然后申请成功后，就看到有了 Client ID 与 Client Secret 的一个页面，页面下边就是申请刚填写的的信息，均可以再次编辑！ 只有 Client ID 与 Client Secret 是在配置 Hexo 的时候需要用。

![申请成功](https://github.com/zhwangart/gitalk/raw/master/source/OAuth-application.png)

看见好多网友做截图时候把 Client ID 与 Client Secret 打码，我有一种木有必要的感觉...

### 在 Hexo 中引入 Gitalk

这个看 [Gitalk](https://github.com/gitalk/gitalk) 里面的引入方法就可以了，如果你使用 Ocean 的话，那么忽略这里！

``` html
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
```

### 创建一个仓库

在 GitHub 中 创建一个仓库用来存放评论，因为 Gitalk 的评论其实是在 GitHub 仓库自动创建 Issue 

### 用法

这个一定要**仔细**，不管是创建一个 comment.ejs 或者 gitalk.ejs 模板文件，还是直接在 post.ejs 模板页中写入，用法都是一样的。因为 Gitalk 是在文章的最底部出现，所以我不选择在 head 中引入 css 与 js ，当然这么做可能不是很规范。

> 因为主题 Ocean 是用 ejs 写的，所以 ejs 为例，创建一个 gitalk.ejs 代码如下

``` xml
<% if (theme.gitalk.enable) { %>  // 这里判断主题是否开启评论
  <div id="gitalk-container"></div>  // 盛放 Gitalk 的容器
  <%- css('https://unpkg.com/gitalk/dist/gitalk.css') %>
  <%- js('https://unpkg.com/gitalk/dist/gitalk.min.js') %>
  <script type="text/javascript">
      var gitalk = new Gitalk({
        clientID: '<%- theme.gitalk.clientID %>',  // 这里一定要注意主题 config.yml 中 clientID 的大小写，否则参数传不过来，对应下边的 "问题一"
        clientSecret: '<%- theme.gitalk.clientSecret %>',
        repo: '<%- theme.gitalk.repo %>',
        owner: '<%- theme.gitalk.owner %>',
        admin: ['<%- theme.gitalk.admin %>'],
        id: location.pathname,      // 保持默认，官方注释（Ensure uniqueness and length less than 50），文章 URL 不能太长 "问题二"
        distractionFreeMode: false  // 不喜欢评论时候的遮盖层所以选择 false ，而且我觉得这个也没有必要放在 config.yml 中配置
      })

  gitalk.render('gitalk-container')
  </script>
<% } %>
```

在 article.ejs（模板页名称因主题而异）中 include gitalk.ejs 因为我只需要在 post 中加入评论功能其他页面不需要所以做了 post 判断！

``` xml
<% if (is_post()) { %>
  <%- partial('post/gitalk') %>
<%} %>
```

然后在主题 config.yml 中添加配置参数，注意 `repo` **只需要写名称**。

``` yml
# Gitalk
gitalk:
  enable: true
  clientID: b056dd67656dd67522d6  # 换成你申请 GitHub Application 网页上对应的 Client ID 与 Client Secret 参数
  clientSecret: 05c56dd6736f12ac156dd6711956dd67e156dd67  # 同上
  repo: gialk  # 换成你创建的仓库，首先确保该仓库已经创建，其次只需要写名称，比如 "gialk"，否则 "问题三"
  owner:   # 你的 Github ID
  admin:   # 你的 Github ID 就可以，官方注释（Facebook-like distraction free mode）说明还可以添加其他有权限的人
```

## 报错与解决方案

### 问题一

> 未找到相关的 Issues 进行评论 请联系 @xxxxxx 初始化创建。

[](./source/Not-found-issues.png)

这个问题分为两种情况：

1. 确实是没有初始化创建，那么点击 "使用 Github 登录" 一次就好了！
2. 点击 "使用 Github 登录" 跳转到了 404 页面，那么就很有可能是 Client ID 与 Client Secret 的参数没传过来，右键检查页面是否有参数，如果为空的话，检查 ClientID（注意 Client 首字母大小写） 与 ClientSecret 这两个字段是否与 config.yml 中的一致。 

### 问题二

> Error: Validation Failed.

[](./source/Error-Validation-Failed.png)

在 Gitalk 的 [Issues](https://github.com/gitalk/gitalk/issues/102) 找到原因：文章的 URL 过长，生成 issue 时超过了 Label 的长度限制，这的注释也写的很明确：`id: location.pathname,      // Ensure uniqueness and length less than 50` 。有两种解决方案：

1. 按照 `Ensure uniqueness and length less than 50` 要求，精简文章名称，避免使用中文，缩短 URL...
2. 文章名经 URL 编码后转 MD5 [JavaScript-MD5](https://github.com/blueimp/JavaScript-MD5)。

``` xml
// 引入 JavaScript-MD5
<%- js('https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js') %>

id: location.pathname,      // Ensure uniqueness and length less than 50
// 改为：
id: md5(location.pathname),
```


### 问题三

> Error: Not Found.

[](./source/Error-Not-Found.png)

很简单就是 repo 写错了！**这里只需要写仓库名称，不要链接！**