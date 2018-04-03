---
title: 搭建可在不同电脑写作和同步的Hexo博客
date: 2018-04-03 14:16:04
tags: 环境搭建
categories: Objective-C
---

## 搭建可在不同电脑写作和同步的Hexo博客

1. 安装`Node.js` 和 `git`环境，具体参照[Hexo文档](https://hexo.io/zh-cn/docs/index.html)
2. 在Github上创建仓库 `leoliuyt.github.io`
3. 创建两个分支：`master` 与 `hexo`
3. 设置`hexo`为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）
4. 使用`git clone https://github.com/leoliuyt/leoliuyt.github.io.git`拷贝仓库
5. 在本地`leoliuyt.github.io`文件夹下通过 依次执行`hexo init Blog`、`cd Blog`、`npm install` 和 `npm install hexo-deployer-git --save`（此时当前分支应显示为hexo）
6. 修改`_config.yml`中的`deploy`参数，分支应为`master`
7. 依次执行`git add .`、`git commit -m “…”`、`git push origin hexo`提交网站相关的文件；
8. 执行`hexo generate -d`生成网站并部署到GitHub上。

这样一来，在GitHub上的`leoliuyt.github.io`仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。

## 写博客及发布同步

```
//首先进入Blog 目录
cd /Users/lbq/Documents/Code/LL_Workspace/leoliuyt.github.io/Blog

//创建博客
hexo new 搭建可在不同电脑写作和同步的Hexo博客

//生成博客及发布
hexo g -d

//本地测试使用
hexo g
hexo s

//发布成功后返回到git目录
cd ..

//git提交
git add .
git commit -m "提交"
git push origin hexo

//清除缓存使用
hexo clean
```

## 主题配置

修改站点目录下`_config.yml` 中的 `theme: next`

`next`主题的配置参考[NexT](https://theme-next.iissnan.com/getting-started.html)

## hexo下新建页面下如何放多个文章？

1. `hexo new page categories` //常见一个新的page
2. 在主题配置文件`_config.yml`中的menu 添加 `Objective-C: /categories/Objective-C/ || th`
3. 在博客的markdown文件中添加 `categories: Objective-C`

## 参考

[Hexo文档](https://hexo.io/zh-cn/docs/index.html)
[在不同电脑上进行Hexo写作与同步](https://leroyli.github.io/2016/11/07/hexo-more-PC/)
[GitHub Pages + Hexo搭建博客](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo搭建博客/#more)
[NexT](https://theme-next.iissnan.com/getting-started.html)
[hexo下新建页面下如何放多个文章？](https://www.zhihu.com/question/33324071)

