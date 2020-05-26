---
title: 导出TypeScript格式的api到项目
date: 2019-12-30 14:30:01
tags: ts node
---

## 背景

最近项目往TypeScript方向重构，前端自己写的东西比较容易定义类型的，问题在于后端给的api数据无法定义类型，很多情况只能用any大法略过。幸好公司内部的api文档使用YApi搭建的，这个框架提供了文档导出的接口，接口包含了完整的数据类型的定义，这些定义取决于后端同事对api文档的维护程度，所以做这个导出api类型到项目中的好处不仅仅可以使整个项目比较完整严谨，还可以反推后端同事更好的对api文档进行维护。

<a name="b65fP"></a>
## 操作步骤

<a name="zUYw5"></a>
### 1. 先获取api文档的JSON数据

官方提供了以下的几个接口：

``` javascript
/api/open/run_auto_test [运行自动化测试]
/api/open/import_data [导入数据]
/api/interface/add [新增接口]
/api/interface/save [保存接口]
/api/interface/up [更新接口]
/api/interface/get [获取接口]
/api/interface/list [获取接口列表]
/api/interface/list_menu [获取接口菜单]
/api/interface/add_cat [新增接口分类]
/api/interface/getCatMenu [获取所有分类]
```

刚开始的做法：

- 先在本地获取接口列表；
- 然后遍历列表去逐个获取接口；

总共44个接口数据，大概耗时2min左右；

通过阅读源码，发现有一个可以获取所有接口数据的api【/plugin/export】，这样就方便很多了，直接拿到所有数据然后格式化即可。

<a name="UcH37"></a>
### 2. 每个接口生成对应数据格式文件

通过node，写脚本命令去跑，每个接口对应一个文件。

这里有个小插曲，为了防止接口命名重复，刚开始是通过api的路径去生成对应的接口名称，其中里面包含的复杂类型用当前key值拼接上父级的变量名。

```javascript
// api test1的路径为 /api/aaa/bbb/ccc
// api test2的路径为 /api/ddd/bbb/ccc

// 生成的interface命名

// test1文件
interface ApiAaaBbbCccReq { // 请求数据类型结构
	uid: string;
  test?: string;
  ...
}

interface ApiAaaBbbCccRes { 响应数据类型结构
	status: number;
  content: ApiAAABBBCCCContent
}

interface ApiAaaBbbCccContent {
	text: string;
  itemList: ApiAaaBbbCccContentItemList;
}

interface ApiAaaBbbCccContentItemList {
	...
}
  
// test2
  
interface ApiDddBbbCccReq {
	...
}
...
```

这样情况显而易见就是interface名会很长，看起来跟用起来会很不舒服，虽然打包的时候会删掉这些，但在写的过程中仍旧会引用到这么长的类型。

所以，最终完善成以下的方式。

```javascript
// api test1的路径为 /api/aaa/bbb/ccc
// api test2的路径为 /api/ddd/bbb/ccc

// 生成的interface命名

// test1文件

// 命名取路径后两位
// 因为这些接口类型会以同一个出口的方式引入，所以为了防止命名冲突，使用命名空间包裹
declare namespace BbbCcc { 
  interface Request {
    uid: string;
  	test?: string;
  	...
  }
   interface Response {
     status: number;
     content: Content
   }
   interface Content {
     text: string;
     itemList: Content.ItemList;
   }
   namespace Content {
		 interface ItemList {
     	...
     }
	 }
}
  

  
// test2文件
  
declare namespace ApiDddBbbCcc { // 这部分做了处理，如果重名，则会用的完整的链接作变量名
	...
}
```

<a name="d6ydV"></a>
### 3. 统一导出方法及类型

```javascript
// 在api 中的 index.js下导出所有方法及类型

// 在此使用三斜线导入所有的类型，在页面中引用方法的同时，类型也已经引用好了
/// <reference path="./apiList/BbbCcc.d.ts" />
/// <reference path="./apiList/ApiDddBbbCcc.d.ts" />

import { postRequest } from '../utils/Request'; // ajax请求方法
import config from '../utils/config'; // 静态配置

export function $BBBCCC (data: BbbCcc.Request, loading: boolean = true): Promise<BbbCcc.Response> {
    return postRequest(`${config.baseurl}/api/aaa/bbb/ccc`, data, loading)
}

export function $ApiDddBbbCcc (data: ApiDddBbbCcc.Request, loading: boolean = true): Promise<ApiDddBbbCcc.Response> {
    return postRequest(`${config.baseurl}/api/ddd/bbb/ccc`, data, loading)
}
```

<a name="klsoD"></a>
## 感受

在改造完后，顺手迁移了个之前写的供能，最大的直观感受是太舒服了，因为vscode对typscript支持十分的友好。之前还需要去查找后端返回接口的类型，十分繁琐，或者直接用any带过，很不美观，现在直接从头到尾写ts毫无压力啊！！！
