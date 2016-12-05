---
title: ionic2杂记
date: 2016-12-02 21:01:17
categories:
 - 编程
tags:
- Typescript
- ionic2
---

记录一些个人使用ionic2过程中的感受和碰到的问题。
<!-- more -->

## 1 技能链

> Typescript --> Angular2 --> ionic2

ionic2是基于Angular2开发，Angular2是使用Typescript重新改写的。Typescript是Javascript类型的超集。学习之前先列出相关中文文档。

- [Typescript中文网](http://www.tslang.cn/)
- [Angular2中文文档](https://angular.cn/docs/ts/latest/quickstart.html)
- [ionic2](https://yanxiaodi.gitbooks.io/ionic2-guide/content/)

## 2 Typescript

和Javascript相比，在函数、接口、类、泛型都有很大的变化，总的来说，会使得程序更加明确，加强了面向对象的一些设计规范。

interface是接口，class是类。类可以实现接口。

## 3 第三方js库与d.ts声明文件

对于现有第三方js库，根据不同情况采取不同的策略。

- 下载d.ts声明文件
- 自己编写d.ts声明文件
- 实现对应TypeScript的版本

一些常用库诸如`underscope.js`可在github下载到对应的d.ts文件。

[中国家庭称谓计算器](https://github.com/mumuy/relationship)，该API项目逻辑复杂，但是接口只有一个，可以自己编写。以下是本人编写的相应d.ts文件。

```javascript
interface RelationshipOptions {
	text:string;
	sex:number;
	type:string;
	reverse?:boolean;
}
declare function relationship(relationshipOptions:RelationshipOptions):string;
export = relationship;
```

使用方式

```javascript
/// <reference path="relationship.d.ts" />
import relationship = require('./relationship');

let s = relationship({
	text:'爸爸的弟弟',
	sex:1,
	type:'default',
	reverse:true
})

console.log(s); //侄子
```

而项目引用的另一个库[《1900年至2100年公历、农历互转Js代码》](http://blog.jjonline.cn/userInterFace/173.html)，API也比较简单，但ionic1项目中对其进行了比较多的扩展（日期加减），因此采用重写自己的Typescript版本。项目地址 https://github.com/kinegratii/ts-calendars。

## 4 版本与开发IDE

目前ionic2版本为rc3，还未发布正式版，但按照版本规律，许多API已经稳定下来了，不像Beta的时候有项目布局那样大的改变，因此可以开始使用和迁移了。

习惯了JetBrain系列的IDE，因此使用WebStorm进行开发，当然是用VS Code也是大家极力推荐的。


## 5 创建项目

ionic2和ionic1相比，项目布局变化比较多，决定重新创建一个项目，不在原有的项目进行更改，使得一开始就符合ionic2的开发规范。

使用下面的命令行创建一个ionic2项目。

```
ionic start FamilyApp --v2
```

默认使用tabs界面，创建了三个页面。

## 6 页面

使用以下命令创建新的页面。

```
ionic generate page more
```

在`src/pages/`创建以下三个文件。

- `more/more.html` 模板文件
- `more/more.scss` css样式
- `more/more.ts` 组件代码

并且将新页面加入模块`app.module.ts`文件中。

```javascript
@NgModule({
  declarations:[
    # ...
    MorePage,
  ],
  entryComponents:[
    # ...
    MorePage,
  ]
})
```

## 7 导航

习惯先创建所有的页面，完成跳转逻辑，最后再写各个页面的逻辑。

ionic2的导航类似于Android，像一个简单的栈，可以进行push和remove操作。不使用url来决定导航，当然也可以这么做。

`no component factory found for AboutPage`

在刚使用导航（MorePage -> AboutPage）时出现了这个错误，需要将AboutPage加到`app.module.ts`的`declarations`和`entryComponents`，两个都要添加。
