# Gitalk

---
title: Gitalk
author: zhwangart
tags:
  - hexo
  - Github
  - Gitalk
date: 2018-12-06 15:11:49
---

由于我比较笨还英文严重不好，所以按照 [Gitalk](https://github.com/gitalk/gitalk) 提供的文档给 Hexo 添加评论功能，简直就是不能行！各种报错后，翻看 Issues、Google、百度... 用了一天时间终于 Gitalk 可用了。
这里我分享一下我的添加过程与报错的解决方法，供参考借鉴！
<!--more-->


### 申请 GitHub Application

如果有的话在 Github 中 `settings / Developer settings` 选择一个 OAuth App，如果没有 [点击这里申请](https://github.com/settings/applications/new)，当然我第一次使用，显然需要从申请开始。

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

这个看 [Gitalk](https://github.com/gitalk/gitalk) 里面的引入方法就可以了！

``` xml
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
```

### 创建一个仓库

在 GitHub 中 创建一个仓库用来存放评论，因为 Gitalk 的评论其实是在 GitHub 自动创建 Issue 

### 用法

这个一定要**仔细**，不管是新建一个 comment.ejs 或者 gitalk.ejs 模板文件，还是直接在 post.ejs 模板页中写入，用法都是一样的。因为 Gitalk 是在文章的最底部出现，所以我不选择在 head 中引入 css 与 js ，当然这么放可能不是很规范。

> 因为我的主题 Ocean 是用 ejs 写的，所以 ejs 为例。

``` xml
<% if (theme.gitalk.enable) { %>  // 这里判断主题是否开启评论
  <div id="gitalk-container"></div>  // 盛放 Gitalk 的容器
  <%- css('https://unpkg.com/gitalk/dist/gitalk.css') %>
  <%- js('https://unpkg.com/gitalk/dist/gitalk.min.js') %>
  <script type="text/javascript">
      var gitalk = new Gitalk({
        clientID: '<%- theme.gitalk.clientID %>',  // 这里一定要注意主题 config.yml 中 clientID 的大小写，否则参数传不过来
        clientSecret: '<%- theme.gitalk.clientSecret %>',
        repo: '<%- theme.gitalk.repo %>',
        owner: '<%- theme.gitalk.owner %>',
        admin: ['<%- theme.gitalk.admin %>'],
        id: location.pathname,      // 保持默认
        distractionFreeMode: false  // 不喜欢评论时候的遮盖层所以选择 false ，而且我觉得这个也没有必要放在 config.yml 中配置
      })

  gitalk.render('gitalk-container')
  </script>
<% } %>
```

然后在主题 config.yml 中添加配置参数，注意 `repo` **只需要写名称**。

``` yml
# Gitalk
gitalk:
  enable: true
  clientID:   # 申请 GitHub Application 网页上对应的 Client ID 与 Client Secret 参数
  clientSecret:   # 同上
  repo:   # 创建的仓库，只需要写名称不要链接，比如 "gialk"，一定确保该仓库已经创建
  owner:   # 你的 Github ID
  admin:   # 你的 Github ID 就可以，官方注释（Facebook-like distraction free mode）说明还可以添加其他有权限的人
```