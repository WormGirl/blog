# webpack 4: import() and CommonJs
>本文译自[webpack 4: import() and CommonJs](https://medium.com/webpack/webpack-4-import-and-commonjs-d619d626b655)
## 概述
webpack 4中的一个重大变化是通过import()函数引入非ES6模块（即CommonJS模块），
实际上在使用import()时还有很多需要注意的地方，我们先从以下几个名词解释开始说起：
+ Source：包含import()方法的模块
+ Target：import()方法所引用的目标模块
+ non-ESM：没有设置__esModule为true的Commonjs或AMD模块
+ transpiled-ESM：设置__esModule为true的Commonjs模块（是由ES6模块转化过来的）
+ ESM：通常所说的ES6模块
+ strict-ESM：相对更严格的ES6模块(如nodejs的ES6模块)
+ JSON：json文件
## 需要考虑以下情况
+ (A) Source: non-ESM, transpiled-ESM 或 ESM
+ (B) Source: strict-ESM (mjs)
+ (1) Target: non-ESM
+ (2) Target: transpiled-ESM (__esModule)
+ (3) Target: ESM or strict-ESM (mjs)
+ (4) Target: JSON
### 看了以下demo或许更容易理解：
```js
// (A) source.js
import("./target").then(result => console.log(result));
// (B) source.mjs
import("./target").then(result => console.log(result));
// (1) target.js
exports.name = "name";
exports.default = "default";
// (2) target.js
exports.__esModule = true;
exports.name = "name";
exports.default = "default";
// (3) target.js or target.mjs
export const name = "name";
export default "default";
// (4) target.json
{ name: "name", default: "default" }
```
## 先从一个简单的例子说起
### A3 and B3: import(ESM)
>ESM规范实际上已经涵盖了这种情况。
列子里的（A）或（B）中通过import()方法引入（3）将会返回一个带命名空间（namespace）的对象，为了考虑到兼容性我们在命名空间（namespace）对象里引入了__esModule属性使其可被import()导入
（A）或（B）打印的结果如下：
```js
{ __esModule: true, name: "name", default: "default" }
```
### A1: import(CommonJS)
当我们import一个Commonjs模块时，webpack 3 仅仅是解析为module.exports的值，而webpack 4会为importd的commonjs模块创造一个带命名空间的对象，以此来让import()导入的结果统一为带命名空间的对象

CommonJs模块的默认导出始终是module.exports的值，webpack还允许通过import { property } from "commonjs"从commonjs模块中导入某个属性，所以我们也允许import()方法也可以这样做。
> 注意：在这种情况下，默认情况下defalut属性被隐藏在webpack生成的default对象里
```js
// webpack 3
{ name: "name", default: "default" }
// webpack 4
{ name: "name", default: { name: "name", default: "default" } }
```
### B1: 在nodejs .mjs(es6模块)文件中import(CommonJS)
在strict-ESM中我们不允许通过import仅导入某个属性，只允许导入non-ESM的默认导出值
```js
{ default: { name: "name", default: "default" } }
```
### A2: import(transpiled-ESM)
webpack支持通过import()导入已经转换为ES6模块的commonjs模块
```js
{ __esModule: true, name: "name", default: "default" }
```
### B2: 在nodejs .mjs(es6模块)文件中import(transpiled-ESM)
在strict-ESM中不支持__esModule这样的标记，你可以把它理解为一种破坏行为，但至少与node.js一致
```js
{ default: { __esModule: true, name: "name", default: "default" } }
```
### A4 and B4: import(json)
import()导入json文件支持只导入部分属性，包括在strict-ESM中，json也会暴露完整的对象作为默认导出
```js
{ name: "name", default: { name: "name", default: "default" } }
```
所以总结下来只有一种情况发生了改变，导出对象时没有任何问题，有问题的是导出的不是对象，如：
```js
module.exports = 42;
```
你应该这样写：
```js
// webpack 3中
42
// webpack 4中
{ default: 42 }
```
