---
title: jQuery对表单的操作及更多应用
date: 2017-05-26 12:12:57
categories: 
- web前端
tags:
- jQuery
- 表格
- 表单验证
---

![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[IT.BOB](https://itrhx.blog.csdn.net/ "IT.BOB") ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newUpTime2.png) 已于 2022-11-26 17:47:25 修改

___

[GitHub](https://so.csdn.net/so/search?q=GitHub&spm=1001.2101.3001.7020) 项目的README.md为自述文件，可对该项目进行介绍，解释等。

使用 Github Pages 和 [Hexo](https://so.csdn.net/so/search?q=Hexo&spm=1001.2101.3001.7020) 搭建的博客，如果在最开始建立仓库的时候没有创建README.md文件，那么在后期如何添加呢？

添加方法：在根目录 source 文件夹下新建README.md即可，该文件使用 Markdown 语法。

然而当我们执行 `hexo g -d` 部署博客的时候会发现README.md变成了 README.html，如下所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190614161916848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NzU5MjI0,size_16,color_FFFFFF,t_70)  
原因就在于当我们在执行 `hexo g -d` 命令时，.md 文件会被转化成 HTML 文件，并将这些文件放到 public 目录里，最后再提交到远程 GitHub 仓库，而 Hexo 也提供了一个方法，让md文件不被转换成HTML，在根目录的 \_config.yml 配置文件里，找到 skip\_render 关键字，添加 README.md，让解释器跳过渲染就行了：

```
skip_render: README.md
```

官方文档：https://hexo.io/docs/configuration  
拓展阅读：[《Hexo + Github Pages 自定义一个不使用主题模板渲染的独立页面》](https://blog.csdn.net/qq_36759224/article/details/90320295)  
我的博客：[https://www.itbob.cn/](https://www.itbob.cn/)