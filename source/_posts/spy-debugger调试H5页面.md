---
title: spy-debugger调试H5页面
date: 2019-09-03 16:48:27
tags: 调试
---

今天看到看到一个H5的网页效果，觉得很不错，想看看怎么实现的，无奈是运行在微信的的内置浏览器中，并且需要微信验证，无法直接通过url跳转链接，网上找了些方法去绕过"请在微信客户端打开链接"这一限制。

<a name="8aoq6"></a>
## 1. 修改浏览器的User Agent

[修改User Agent方法](https://zhuanlan.zhihu.com/p/39856170)

但是，这种方法在对于open.weixin.qq.com...开头的是不起效的，只能针对通过js对 `MicroMessenger`字段做拦截的情况。

<a name="HaoRl"></a>
## 2. spy-debugger调试

一站式页面调试、抓包工具。远程调试任何手机浏览器页面，任何手机移动端webview（如：微信，HybridApp等）。支持HTTP/HTTPS，无需USB连接设备。  

[官方教程](https://github.com/wuchangming/spy-debugger)

<a name="yY3Du"></a>
### 踩坑1（针对ios）
**<br />**![image.png](https://cdn.nlark.com/yuque/0/2019/png/454049/1567495382520-d6b663c9-272c-4ee8-b783-8354bab41088.png#align=left&display=inline&height=397&name=image.png&originHeight=397&originWidth=1027&size=72330&status=done&width=1027)**<br />**<br />如果按照官方教程来，第四步死活打不开官方所给的链接。

<a name="HSZvy"></a>
#### 解决方法

1. 先打开 跑完 spy-debugger命令后的网页，用相机扫描所给的二维码，并用safari浏览器打开；

![微信图片_20190903163818.png](https://cdn.nlark.com/yuque/0/2019/png/454049/1567500116590-4b3b70a0-f5f7-4b93-98cf-40860950ce2e.png#align=left&display=inline&height=395&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190903163818.png&originHeight=637&originWidth=709&size=30983&status=done&width=440)<br />
<br />



2. 修改safari打开后的url链接，把127.0.0.1改成本机的ip；

![微信图片_20190903164416.png](https://cdn.nlark.com/yuque/0/2019/png/454049/1567500270646-f8b2a155-29b2-4f38-90ab-68b893991811.png#align=left&display=inline&height=170&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190903164416.png&originHeight=170&originWidth=299&size=6817&status=done&width=299)<br />

3. 可以正常显示证书下载页面；

![微信图片_20190903163829.jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/454049/1567500327719-173c89c4-759c-49cd-96c4-d94cd2c62c4b.jpeg#align=left&display=inline&height=363&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190903163829.jpg&originHeight=1334&originWidth=750&size=62175&status=done&width=204)

4. 然后下载安装；

<a name="sRNJy"></a>
### 踩坑2（针对ios）

如果按官方教程来，进行到最后，仍会有如下的提示：


![微信图片_20190903153130.jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/454049/1567495907856-16aac12b-f0da-4b69-926c-63f2892f6191.jpeg#align=left&display=inline&height=404&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190903153130.jpg&originHeight=1334&originWidth=750&size=58870&status=done&width=227)<br />
<br />这是因为没有下载的证书完全信任导致。<br />

<a name="9nOzz"></a>
#### 解决方法：

<br />**通用 => 关于本机 => 证书信任设置 ，然后选择对应的证书信任即可。**<br />**<br />

<a name="woeJW"></a>
### 效果

<br />
<br />**![微信图片_20190903153844.jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/454049/1567496339837-2b1892ca-8c21-429f-b545-28f55ecb0fb1.jpeg#align=left&display=inline&height=377&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190903153844.jpg&originHeight=1334&originWidth=750&size=77837&status=done&width=212)**<br />**（****手机端）**<br />**<br />**<br />**<br />**![1567496107(1).jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/454049/1567496132658-232a2737-7037-4898-a3eb-3cd5e4e2e367.jpeg#align=left&display=inline&height=711&name=1567496107%281%29.jpg&originHeight=711&originWidth=1899&size=111514&status=done&width=1899)**<br />**（PC端)**<br />**<br />**<br />最后即可愉快的调试了~

[语雀地址](https://www.yuque.com/u283460/kb/bvkv4h)
