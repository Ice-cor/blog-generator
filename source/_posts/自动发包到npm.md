---
title: 自动发包到npm
date: 2020-03-19 18:13:54
tags: 部署 npm
---

自动化测试使用的是circle ci，本例全都是以circle配置为准。

<a name="5XN8F"></a>
## yml文件配置
测试的配置不多赘述，主要贴出自动publish的配置；

```yaml
# .circleci/config.yml文件

defaults: &defaults
  docker:
    - image: circleci/node:10
version: 2

...
publish: 
	<<: *defaults
  steps:
    - attach_workspace:
        at: .
    # 自动发布不需要输入账号密码，只需token认证
    - run: npm config set //registry.npmjs.org/:_authToken $NPM_TOKEN 
    - run: npm publish

workflows:
	...
  - publish:
          requires:
            - build
          filters:
            branches:
              # 提交到分支，检测到有分支变动，才自动部署
              only: /deploy/
#            ci 仍旧无法健测到tag，不用此方式自动部署
#            tags:
#              only: /^v[0-9]+(\.[0-9]+)*/
#            branches:
#              ignore: /.*/

```

在此之前，需要做的是：

1. 在自己的npm仓库生成token；
1. 在circle ci找到对应项目，把npm生产的token存到设置中；
> 做上面两步的原因，是因为npm publish 需要输入账号密码，但在ci自动发包的时候不可能将密码放到文件配置中，所以用到了npm token去直接关联；

<a name="bPXXC"></a>
## 本地deploy.sh配置

```bash
#!/bin/env bash

## $1是可传变量
npm version $1 && \
git push

```

<a name="5iuwD"></a>
## 步骤

1. 创建deploy分支
```git
git push origin master:deploy
```

2. 当需要更新npm包版本的时候，把代码同步到deploy分支，即第一步的代码；
2. 本地运行bash脚本 deploy.sh your version，当circle的测试跑完后并且成功后，会自动发包到npm上。


