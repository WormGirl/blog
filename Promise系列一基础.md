# Promise系列一基础

## Promise发展历史

最早的时候promise机制在jQuery和Dojo中是以Deferred API的形式出现。到了2010年，CommonJS项目实现的Promises/A规范日益流行起来。Q和Bluebird等第三方JavaScript promise库也越来越得到社区认可，虽然这些库的实现多少都有些不同。为弥合现有实现之间的差异，2012年Promises/A+组织分叉（fork）了CommonJS的Promises/A建议，并以相同的名字制定了Promises/A+规范。这个规范最终成为了ECMAScript 6规范实现的范本。
 ECMAScript 6增加了对Promises/A+规范的完善支持，即Promise类型。一经推出，Promise就大受欢迎，成为了主导性的异步编程机制。所有现代浏览器都支持ES6promise，很多其他浏览器API（如fetch()和Battery Status API）也以promise为基础。

## promise的状态机制(生命周期)

 promise是一个有状态的对象，可能处于一下3中状态

 1.待定状态（pending）,创建一个新的Promise对象最初是pending状态

 ```js
 console.log(new Promise(() => {}))
 // Promise<pending>
 ```

 2.兑现状态（fulfilled or resolved）,执行resolved方法后会变成resolve状态

 ```js
 let p = new Promise(resolve => resolve())
 console.log(p)
 // Promise<pending>
 setTimeout(() => {
   console.log(p)
   // Promise<resolved>: undefined
 }, 0)
 ```

 3.拒绝状态（rjected）,执行reject方法或者抛出异常后会变成rejected状态

 ```js
 let p = new Promise((resolve,reject) => reject())
 let p2 = new Promise(() => {
   throw new Error('异常了！')
 })
 setTimeout(() => {
   console.log(p1)
   // Promise<rejected>: undefined
   console.log(p2)
   // Promise<rejected>: Error: 异常了！
 }, 0)
 ```

 只要从待定转换为兑现或拒绝，promise的状态就不能再改变，重复调用不会有任何改变。而且，也不能保证promise必然会脱离待定状态。因此，组织合理的代码无论promise解决（resolve）还是拒绝（reject），甚至永远处于待定（pending）状态，都应该具有恰当的行为。而且promise的状态是私有的，不能直接通过JavaScript检测到。这主要是为了避免根据读取到的promise状态，以同步方式处理promise对象。另外，promise的状态也不能被外部JavaScript代码修改。这与不能读取该状态的原因是一样的：promise故意将异步行为封装起来，从而隔离外部的同步代码。

## Promise.resolve()

使用这个方法可以把任何一个值转化为promise，如果传入的是一个promise，该方法会原封不动返回该promise，是一个幂等操作。需要注意的是如果传入的是一个错误对象，也会将其转换为解决的promise。

```js
let m = Promise.resolve(7)
let p = Promise.resolve(new Error('异常了！'))
setTimeout(() => {
  console.log(m === Promise.resolve(m))
  // true
  console.log(p)
  // Promise<resolved>:Error:异常了！
}, 0)
```

## Promise.reject()

Promise.reject()没有照搬Promise.resolve()的幂等逻辑，如果给它传一个promise对象，则这个promise会成为它返回的拒绝的理由：

```js
  setTimeout(() => {
    Promise.reject(Promise.resolve())
    // Promise <rejected>: Promise <resolved>
  }, 0); 
```

## Promise的异常捕获

```js
try { 
  throw new Error('foo'); 
} catch(e) {
  console.log(e); 
  // Error: foo
} 
try {
  Promise.reject(new Error('bar'));
} catch(e) {
  console.log(e);
}
// Uncaught (in promise) Error: bar
```

第一个try/catch抛出并捕获了错误，第二个try/catch抛出错误却没有捕获到。在例子中，拒绝promise的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队列来处理的。因此，try/catch块并不能捕获该错误。代码一旦开始以异步模式执行，唯一与之交互的方式就是promise的方法。

## promise的实例方法

promise实例的方法是连接外部同步代码与内部异步代码之间的桥梁。这些方法可以访问异步操作返回的数据，处理promise成功和失败的结果，连续对promise求值，或者添加只有promise进入终止状态时才会执行的代码。

1.实现了Thenable接口

在ECMAScript暴露的异步结构中，任何对象都有一个then()方法。这个方法被认为实现了Thenable接口。

```js
class MyThenable {
  then() {}
}
```

ECMAScript的Promise类型实现了Thenable接口。

2.Promise.prototype.then()

该方法为promise实例添加处理程序的主要方法。这个then方法接收最多两个参数：onResolved处理程序和onRejected处理程序。这两个参数都是可选的，如果提供的话，则会在promise分别进入“兑现”和“拒绝”状态时执行。

因为promise只能转换为最终状态一次，所以这两个操作一定是互斥的。而且，传给then()的任何非函数类型的参数都会被静默忽略。如果想只提onRejected参数，那就要在onResolved参数的位置上传入undefined。这样有助于避免在内存中创建多余的对象。

需要注意的是该方法支持链式操作，所以该方法会继续返回一个promise，而且返回的是一个新的promise。

```js
  let p1 = new Promise(() => {});
  let p2 = p1.then();
  setTimeout(() => {
    console.log(p1 === p2)
    // false
  }, 200);
```

 这个新promise实例基于onResovled处理程序的返回值构建。换句话说，该处理程序的返回值会通过Promise.resolve()包装来生成新promise。如果没有提供这个处理程序，则Promise.resolve()就会包装上一个promise解决之后的值。如果没有显式的返回语句，则Promise.resolve()会包装默认的返回值undefined。

 ```js
 let p1 = Promise.resolve('foo');
 // 若调用then()时不传处理程序，则原样向后传
 let p2 = p1.then();
 setTimeout(() => {
   console.log(p2)
   // Promise <resolved>: foo
 }, 0);
// 这些都一样
let p3 = p1.then(() => undefined);
let p4 = p1.then(() => {});
let p5 = p1.then(() => Promise.resolve());
setTimeout(() => {
  console.log(p3)
  // Promise <resolved>: undefined
}, 0);

setTimeout(() => {
  console.log(p4)
  // Promise <resolved>: undefined
}, 0);

setTimeout(() => {
  console.log(p5)
// Promise <resolved>: undefined
}, 0);
 ```

如果有显式的返回值，则Promise.resolve()会包装这个值：

 ```js
 // 这些都一样
 let p6 = p1.then(() => 'bar');
 let p7 = p1.then(() => Promise.resolve('bar')); setTimeout(console.log, 0, p6); 
 // Promise <resolved>: bar
 setTimeout(console.log, 0, p7);
 // Promise <resolved>: bar
 // Promise.resolve()保留返回的promise
 let p8 = p1.then(() => new Promise(() => {}));
 ```

抛出异常会返回拒绝的promise：

```js
let p10 = p1.then(() => { throw 'baz'; });
// Uncaught (in promise) baz
setTimeout(console.log, 0, p10);
// Promise <rejected> baz
```

注意，返回错误值不会触发上面的拒绝行为，而会把错误对象包装在一个解决的promise中：

```js
let p11 = p1.then(() => Error('qux'));
setTimeout(console.log, 0, p11);
// Promise <resolved>: Error: qux
```

onRejected处理程序也与之类似：onRejected处理程序返回的值也会被Promise.resolve()包装。乍一看这可能有点违反直觉，但是想一想，onRejected处理程序的任务不就是捕获异步错误吗？因此，拒绝处理程序在捕获错误后不抛出异常是符合promise的行为，应该返回一个解决promise。
下面的代码片段展示了用Promise.reject()替代之前例子中的Promise.resolve()之后的结果：

```js
let p1 = Promise.reject('foo');
// 调用then()时不传处理程序则原样向后传
let p2 = p1.then();
// Uncaught (in promise) foo
setTimeout(() => {
  console.log(p2)
  // Promise <rejected>: foo
}, 0);

// 这些都一样
let p3 = p1.then(null, () => undefined);
let p4 = p1.then(null, () => {});
let p5 = p1.then(null, () => Promise.resolve());
console.log(p3);
// Promise <pending>
setTimeout(() => {
  console.log(p3);
  // Promise <resolved>: undefined
}, 0)
```

3.Promise.prototype.catch()

catch方法用于给promise添加拒绝处理程序。这个方法只接收一个参数：onRejected处理程序。实际上，这个方法就是一个语法糖，调用它就相当于调用Promise.prototype.then(null, onRejected)。

4.Promise.prototype.finally()

finally()方法用于给promise添加onFinally处理程序，这个处理程序在promise转换为解决或拒绝状态时都会执行。这个方法可以消除onResolved和onRejected处理程序中出现冗余代码。onFinally处理程序没有办法知道promise的状态是解决还是拒绝，所以这个方法主要用于减少代码冗余。finally()方法会返回一个新的promise实例：

```js
let p1 = new Promise(() => {});
let p2 = p1.finally();
setTimeout(() => {
  console.log(p1 === p2)
  // false
}, 0);
```

 这个返回的新promise实例不同于then()或catch()方式返回的实例。onFinally被设计为一个状态无关的方法，所以在大多数情况下它将表现为父promise的传递。对于已解决状态和被拒绝状态都是如此。

 ```js
let p1 = Promise.resolve('foo');
// 这里都会原样后传,返回的promise都是一样的，与finnaly返回值无关
let p2 = p1.finally();
let p3 = p1.finally(() => undefined);
let p4 = p1.finally(() => {});
let p5 = p1.finally(() => Promise.resolve());
let p6 = p1.finally(() => 'bar');
let p7 = p1.finally(() => Promise.resolve('bar'));
let p8 = p1.finally(() => Error('qux'));
setTimeout(() => {
  console.log(p2)
  // Promise <resolved>: foo
}, 0);
 ```

 需要注意的是如果返回的是一个待定的promise，或者onFinally处理程序抛出了错误（显式抛出或返回了一个拒绝promise），则会返回相应待定或拒绝的promise，如下所示：

 ```js
 // Promise.resolve()保留返回的promise
 let p9 = p1.finally(() => new Promise(() => {}));
 let p10 = p1.finally(() => Promise.reject());
 // Uncaught (in promise): undefined
setTimeout(() => {
  console.log(p9)
  // Promise <pending>
}, 0);
setTimeout(() => {
  console.log(p10)
  // Promise <rejected>: undefined
}, 0);
let p11 = p1.finally(() => { throw 'baz'; });
// Uncaught (in promise) baz
setTimeout(() => {
  console.log(p11)
  // Promise <rejected>: baz
}, 0, p11);
 ```
