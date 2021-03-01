# new 运算符

> new constructor[([arguments])]
constructor为class或者函数

## new 关键字会进行如下的操作

1. 创建一个空的简单JavaScript对象（即{}）；
2. 设置该对象的constructor为new操作符的构造函数；
3. 将步骤1新创建的对象作为this的上下文 ；
4. 如果该函数没有返回对象，则返回`this`；

> new a等价与new a(),没有参数的情况下可以这么写

## 箭头函数注意

1. 箭头函数作用域如何绑定的？
2. 箭头函数不能用作构造函数，所以箭头函数没有自己的`new.target`，如

    ```js
    let a = () => {}
    new a() // error
    ```

3. 箭头函数没有prototype属性，所以也没有自己的`super`
4. 箭头函数不能使用自己的`arguments`

## 函数默认参数值问题

函数参数按默认顺序初始化的，所以后定义默认值参数可以使用先定义的参数，且参数初始化顺序遵循暂时性死区规则

```js
function foo(name = 'worm', nickName = name){
// 合法
}
```

## 如何让函数递归与函数名字解耦

使用arguments.callee

```js
function foo (num) {
  if (num < 1) {
    return 1
  } else {
    return num * arguments.callee(num - 1)
  }
}
```

严格模式下使用`arguments.callee`会报错，可以使用函数表达式

```js
let otherFoo = function foo (num) {
  if (num < 1) {
    return 1
  } else {
    return num * foo(num - 1)
  }
}
```

## 简单类型/原始数据类型

undefined、null、boolean、number、string、symbol