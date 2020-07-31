# 前言

在做下面👇的题目之前，我希望你能清楚几个知识点。

(如果你感觉一上来不想看这些列举的知识点的话，直接看后面的例子再来理解它们也可以)

**`event loop`它的执行顺序：**

-   一开始整个脚本作为一个宏任务执行
-   执行过程中同步代码直接执行，宏任务进入宏任务队列，微任务进入微任务队列
-   当前宏任务执行完出队，检查微任务列表，有则依次执行，直到全部执行完
-   执行浏览器UI线程的渲染工作
-   检查是否有`Web Worker`任务，有则执行
-   执行完本轮的宏任务，回到2，依此循环，直到宏任务和微任务队列都为空

**微任务包括：**`MutationObserver`、`Promise.then()或catch()`、`Promise为基础开发的其它技术，比如fetch API`、`V8`的垃圾回收过程、`Node独有的process.nextTick`。

**宏任务包括**：`script` 、`setTimeout`、`setInterval` 、`setImmediate` 、`I/O` 、`UI rendering`。

**注意**⚠️：在所有任务开始的时候，由于宏任务中包括了`script`，所以浏览器会先执行一个宏任务，在这个过程中你看到的延迟任务(例如`setTimeout`)将被放到下一轮宏任务中来执行。





# 1. 基础题

## 1.1

```js
const promise1 = new Promise((resolve, reject) => {
  console.log('promise1')
})
console.log('1', promise1);
```

过程分析：

-   从上至下，先遇到`new Promise`，执行该构造函数中的代码`promise1`
-   然后执行同步代码`1`，此时`promise1`没有被`resolve`或者`reject`，因此状态还是`pending`

结果：

```js
'promise1'
'1' Promise{<pending>}
```



## 1.2

```js
const promise = new Promise((resolve, reject) => {
  console.log(1);
  resolve('success')
  console.log(2);
});
promise.then(() => {
  console.log(3);
});
console.log(4);
```

过程分析：

-   从上至下，先遇到`new Promise`，执行其中的同步代码`1`
-   再遇到`resolve('success')`， 将`promise`的状态改为了`resolved`并且将值保存下来
-   继续执行同步代码`2`
-   跳出`promise`，往下执行，碰到`promise.then`这个微任务，将其加入微任务队列
-   执行同步代码`4`
-   本轮宏任务全部执行完毕，检查微任务队列，发现`promise.then`这个微任务且状态为`resolved`，执行它。

结果：

```js
1 2 4 3
```



## 1.3

```js
const promise = new Promise((resolve, reject) => {
  console.log(1);
  console.log(2);
});
promise.then(() => {
  console.log(3);
});
console.log(4);
```

过程分析

-   和题目二相似，只不过在`promise`中并没有`resolve`或者`reject`
-   因此`promise.then`并不会执行，它只有在被改变了状态之后才会执行。

结果：

```js
1 2 4
```



## 1.4

```js
const promise1 = new Promise((resolve, reject) => {
  console.log('promise1')
  resolve('resolve1')
})
const promise2 = promise1.then(res => {
  console.log(res)
})
console.log('1', promise1);
console.log('2', promise2);
```

过程分析：

-   从上至下，先遇到`new Promise`，执行该构造函数中的代码`promise1`
-   碰到`resolve`函数, 将`promise1`的状态改变为`resolved`, 并将结果保存下来
-   碰到`promise1.then`这个微任务，将它放入微任务队列
-   `promise2`是一个新的状态为`pending`的`Promise`
-   执行同步代码`1`， 同时打印出`promise1`的状态是`resolved`
-   执行同步代码`2`，同时打印出`promise2`的状态是`pending`
-   宏任务执行完毕，查找微任务队列，发现`promise1.then`这个微任务且状态为`resolved`，执行它。

结果：

```js
'promise1'
'1' Promise{<resolved>: 'resolve1'}
'2' Promise{<pending>}
'resolve1'
```



## 1.5

```js
const fn = () => (new Promise((resolve, reject) => {
  console.log(1);
  resolve('success')
}))
fn().then(res => {
  console.log(res)
})
console.log('start')
```

这道题里最先执行的是`'start'`吗 🤔️ ？

请仔细看看哦，`fn`函数它是直接返回了一个`new Promise`的，而且`fn`函数的调用是在`start`之前，所以它里面的内容应该会先执行。

结果：

```js
1
'start'
'success'
```



## 1.6

```js
const fn = () =>
  new Promise((resolve, reject) => {
    console.log(1);
    resolve("success");
  });
console.log("start");
fn().then(res => {
  console.log(res);
});
```

是的，现在`start`就在`1`之前打印出来了，因为`fn`函数是之后执行的。

**注意⚠️**：之前我们很容易就以为看到new Promise()就执行它的第一个参数函数了，其实这是不对的，就像这两道题中，我们得注意它是不是被包裹在函数当中，如果是的话，只有在函数调用的时候才会执行。

答案：

```js
"start"
1
"success"
```

