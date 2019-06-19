---
title: TypeScript渐进(1)-安装及调试工具
date: 2019-06-19 15:05:53
tags: TypeScript
---

## TypeScript安装及工具调试

我是采用vscode作为开发编辑器，往后的TS调试也是基于此编辑器的。

TS的全局安装及调试工具安装命令。

```
npm install typescript -g 
npm install ts-node -g
```

注意记下ts-node安装后的文件路径，后面需要用到。

![复制文件路径](https://i.loli.net/2019/06/19/5d09e5c45e02535213.jpg)

## 调试

1.在vscode上创建tsDemo文件夹

2.在tsDemo文件夹中创建hello.ts文件

3.在文件里写一句话console.log('hello ts!')然后保存

4.Windows 用户注意了，这里需要单独运行一些命令（Linux 用户和 macOS 用户不用执行）

```
npm init -y
npm i -D ts-node typescript
```

5.创建.vscode文件夹，并在文件夹里面创建launch.json文件，里面的内容：

```
{
     "configurations": [
         {
         "name": "ts-node",
         "type": "node",
         "request": "launch",
         "program": "注意看这里，要写成ts-node对应的可执行文件，Windows 用户注意了，你应该写成 ${workspaceRoot}/node_modules/ts-node/dist/bin.js",
         "args": ["${relativeFile}"],
         "cwd": "${workspaceRoot}",
         "protocol": "inspector"
         }
     ]
 }
```

6.打开tsDemo/hello.ts,找到调试选项，选择ts-node,然后点击调试即可看到输出结果

![调试选项](https://i.loli.net/2019/06/19/5d09ea493b23483988.jpg)


![输出结果](https://i.loli.net/2019/06/19/5d09eaddcd3ef55500.jpg)


参考文章: [https://segmentfault.com/a/1190000011935122](https://segmentfault.com/a/1190000011935122)



