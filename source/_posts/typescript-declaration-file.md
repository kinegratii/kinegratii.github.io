---
title: typescript-declaration-file
date: 2017-08-27 18:24:02
categories: 编程
tags:
- Typescript
---


TypeScript声明文件。

<!-- more -->


if you want to use relative path in the import, you will need:

- put the add.d.ts next to add.js
- define the file as an external module:

```
// add.d.ts
declare function add(n1: number, n2: number): number;
export = add;
```

i.e. loose the "declare module "add"" part.

There are two ways to define declarations for a .js module:

1 using `declare module "foo"` and then you can have multiple module definitions in the same file:

```
// mydefinitions.d.ts
declare module "mod1" {
   export var x = 0;
}

declare module "mod2" {
   export var y = 0;
}

declare module "mod3" {
   export var z = 0;
}
```

and consuming them would have to be using absolute names:

```
// main.ts

/// <reference path="myDefintions.d.ts" />
import * as mod1 from "mod1";
import mod2 = require("mod2");
import {z} from "mod3";
```

2 alternatively you can define as a file, where the name of the file is the name of the module

```
// myModule.d.ts

declare var m = 0;
export = m;
```

and consume it as a normal .ts module:

```
import m = require("./myModule");
m.toString();
```
