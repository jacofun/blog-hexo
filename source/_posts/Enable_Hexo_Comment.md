---
title: 使用Waline+MongoDB实现Hexo评论功能
date: 2023-12-31 02:30:01
update: 2024-01-01 22:17:00
index_img: /img/index/Enable_Hexo_Comment.png
tags: [Hexo, Fluid, Programming]
---
历时两天半，博客的评论功能算是终于实现，中间踩了不少的坑，走了不少弯路，简单记录一下部署的流程。
<!--more-->
## 0. 写在前面 ##
我一开始搭建这个博客的目的只是简单记录一下成长经历和学习过程，后来想提升一下博客的交互性，所以就从添加评论开始。

	实现一看似简单的功能实际上一点也不简单。
	 —— 我说的

## 1.实现效果 ##

Waline评论服务及MongoDB数据库均部署在云端，具备弹性伸缩的良好扩展性与移植性，搭配Cloudflare，方便后期配置[WAF(网络应用防火墙)](https://www.cloudflare.com/zh-cn/application-services/products/waf/)。Waline评论具备评论通知功能，在收到评论后，可以选择使用邮件、QQ、微信消息等方式通知博主，这里选择的是使用邮件。

目前，在中国大陆境内三网也能有比较友好的访问延迟（Cloudflare yyds！）

## 1.数据库 ##
这里使用的是[mongodb.com](https://www.mongodb.com/zh-cn)的官方云数据库，可以使用Github账号或Google账号登录。

登录后创建一个Shared方案的数据库，区域可以根据部署类型选择。

独立部署可以选择香港，搭配Cloudflare建议选择美国西部区域

注：中国大陆境内连接Cloudflare的endpoint区域各不相同。一般来说，移动连接到香港，联通和电信连接到美国

然后就是按照步骤添加用户，密码可以使用自动生成的（注意备份）

访问策略要选择anywhere(0.0.0.0/0)

![](/img/post/Enable_Hexo_Comment/database1.png)

到这里数据库就创建完成了。

## 2.部署Waline后端 ##

参照官方文档采用[Vercel部署](https://waline.js.org/guide/deploy/vercel.html)

此外，还要对Waline添加cors策略([什么是跨域资源共享](https://baike.baidu.com/item/%E8%B7%A8%E6%9D%A5%E6%BA%90%E8%B5%84%E6%BA%90%E5%85%B1%E4%BA%AB/22911772?fr=ge_ala))

Vercel平台为我们提供了[添加cors策略](https://vercel.com/guides/how-to-enable-cors)的方法，只需要在vercel.json文件中添加headers即可。可以直接采用以下配置：

```json
{
  "name": "comment",
  "github": {
    "silent": true
  },
  "builds": [
    {
      "src": "index.js",
      "use": "@vercel/node@2.5.10"
    }
  ],
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "index.js"
    }
  ],
   "headers": [
  {
    "source": "/",
    "headers": [
        { "key": "Access-Control-Allow-Credentials", "value": "true" },
        { "key": "Access-Control-Allow-Origin", "value": "*" },
        { "key": "Access-Control-Allow-Methods", "value": "GET,OPTIONS,PATCH,DELETE,POST,PUT" },
        { "key": "Access-Control-Allow-Headers", "value": "X-CSRF-Token, X-Requested-With, Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, X-Api-Version" }
      ]
    }
  ]
}
```
## 3.配置Vercel环境变量 ##

参照[官方的配置](https://waline.js.org/reference/server/env.html)来操作就好，不过我们使用的是MongoDB，具体的配置请参照官方的[多数据库服务支持](https://waline.js.org/guide/database.html)


使用mongodb.com的服务是集群部署，HOST的获取方法见下图：

![](/img/post/Enable_Hexo_Comment/database2.png)

登录数据库管理页面后，选择以Driver形式连接，将Node.js的Version调整为如2.2.12 or later，即可显示集群的三个host地址以及环境变量中所需的各个参数。

填入Vercel平台的环境变量后重新部署即可。

其他好玩的环境变量[参照官方文档](https://waline.js.org/reference/server/env.html)

## 4.配置自定义域名（可选） ##
在Vercel平台中添加一个你自己的域名，然后在Cloudflare上添加CNAME记录，即可实现中转访问Vercel平台


## 5.修改Fluid配置文件及评论客户端##
serverURL就是vercel平台中的地址（或自定义的经cloudflare中转的域名），具体配置见Fluid主题[用户手册](https://hexo.fluid-dev.com/docs/guide/#%E8%AF%84%E8%AE%BA)

Waline的源码没有深入了解过，客户端的简单配置比如说按钮的文字等等[自定义语言](https://waline.js.org/cookbook/customize/locale.html)需要修改`\node_modules\hexo-theme-fluid\layout\_partials\comments\waline.ejs`我使用的样式：

```javascript
<% if (theme.waline.serverURL) { %>
  <div id="waline"></div>
  <script type="text/javascript">
  const locale = {
	  nick: '昵称(必填)',
	  nickError: '昵称太短了~',
	  placeholder: '可以直接评论哦~',
	  sofa: '评论区空空如也~',
	  refresh: '刷新一下~',
	  login: '登录(可选)',
	  submit: '发布',
  }
    Fluid.utils.loadComments('#waline', function() {
      Fluid.utils.createCssLink('<%= url_join(theme.static_prefix.waline, 'waline.min.css') %>')
      Fluid.utils.createScript('<%= url_join(theme.static_prefix.waline, 'waline.min.js') %>', function() {
        var options = Object.assign(
          <%- JSON.stringify(theme.waline || {}) %>,
          {
            el: '#waline',
            path: <%= theme.waline.path %>,
			locale,
          }
        )
        Waline.init(options);
        Fluid.utils.waitElementVisible('#waline .vcontent', () => {
          var imgSelector = '#waline .vcontent img:not(.vemoji)';
          Fluid.plugins.imageCaption(imgSelector);
          Fluid.plugins.fancyBox(imgSelector);
        })
      });
    });
  </script>
  <noscript>Please enable JavaScript to view the comments</noscript>
<% } %>

```

## 6.写在最后 ##
后面的文章会考虑写一些PaaS还有SaaS相关的东西吧。

接下来很长一段时间应该不会再去折腾博客的新功能了，实在有点累，女朋友也没有认真去陪T_T，接下来的日子还是认真生活和工作为主了~

