---
title: Hexo+github+git搭建个人博客
date: 2018-03-02 01:28:48
tags: hexo github
---
捣鼓了半天，终于搭好了博客，换好了皮肤，要开始没羞没臊的博客生活了~
**注意：此篇教程使用gitbase+git命令来操作，小伙伴们使用自带的IDE简化操作流程也是可以的，并且系统中要配置Node.js。**
[Node配置教程](https://www.jianshu.com/p/03a76b2e7e00)  [git操作教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

### 注册一个github账号
1、登陆[github官网](https://github.com/)
2、注册一个github账号 [注册github账号教程](https://www.cnblogs.com/Amedeo/p/7664224.html)
3、创建一个空白仓库，仓库名字设置为 **你的用户名.github.io**
4、关联本地仓库与github远程仓库 [仓库管理教程](https://www.jianshu.com/p/dcbb8baa6e36)

完成以上步骤就已经成功了一大半了~

### Hexo教程
1、在本地仓库安装Hexo

``` bash
npm install -g hexo-cli
hexo init myBlog
cd myBlog
npm i
hexo new 开博大吉 
```
你会看到一个 md 文件的路径

2、编辑开博大吉这个md文件，内容随意

``` bash
start xxx/开博大吉.md 
```
xxx为md文件的路径

**注意：start为window下的命令。**

3、编辑网站配置

``` bash
start _config.yml
```

  **3.1 把第 6 行的 title 改成你想要的名字**

  **3.2 把第 9 行的 author 改成你的大名**

  **3.3 把最后一行的 type 改成 type: git**

  **3.4 在最后一行后面新增一行，左边与 type 平齐，加上一行 repo: 仓库地址（仓库地址为你在github所创建的的网址）**

4、安装 git 部署插件

``` bash
npm install hexo-deployer-git --save
hexo deploy
```

5、进入github对应仓库，打开 GitHub Pages 功能，如果已经打开了，就直接点击预览链接

6、你现在应该看到了你的博客！

个人博客：[钟文钦的博客](https://ice-cor.github.io/)
[Hexo文档](https://hexo.io/zh-cn/docs/)

