# promise

<a name="xPBL0"></a>
### 一、Promise 含义
Promise 是异步编程的一种解决方案，比传统的回调函数和事件更合理。Promise 简单说是一个容器，里面保存着未来才会结束的事件的结果（通常是一个异步操作的结果）。<br />语法上，Promise 是一个对象，从它可以获取异步操作的消息，各种异步操作都可以用同样的方法处理。<br />Promise 对象的特点：<br />(1)对象的状态不受外界影响。 Promise 对象代表一个异步操作，有三种状态：pending（进行中），fulfilled（已成功），rejected（已失败）。只有异步操作结果可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。<br />(2)一旦状态改变，就不会更改，任何时候得到的都是这个结果。Promise 状态改变只有两种可能： 从 pending 变为 fulfilled 和 从 pending 变为 rejected。只要这两种情况发生，状态就不会再变，这时称为 resolved（已定型）。<br />(3)如果改变已经发生，再对 Promise 对象添加回调函数，也会立即得到改变的结果。这与事件的区别是：事件一旦错过，再去监听，等不到结果。<br />(4)Promise 无法取消，一旦新建，就会立即执行，无法中途取消。<br />(5)如果不设置回调函数，Promise 内部抛出的错误不会反应到外部。<br />(6)当处于 pending 状态时，无法得知当前进展到哪一阶段（刚开始还是即将完成）。
<a name="C4iVR"></a>
### 二、基本用法
ES6 规定 Promise 是一个构造函数，用来生成 Promise 实例。
```javascript
const p1 = new Promise( (resolve, reject) => {
	setTimeout( () => reject(new Error('fail')), 3000 );
} );
const p2 = new Promise( (resolve, reject) => {
	setTimeout(() => resolve(p1), 1000);
} );
p2.then(
	result => console.log(result)
).catch(
	error => console.log(error)
);
```
Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 和 reject，它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
<a name="XSoDZ"></a>
#### 1. resolve 和 reject
**(1) resolve**<br />resolve 函数的作用是，将 Promise 对象的状态从 “未完成”变为“成功”（即从 pending 变为 resolved）。**在异步操作成功时调用，并将异步操作的结果，作为参数传递出去。**<br />**(2) reject**<br />reject 函数的作用是，将 Promise 对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected）。**在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。**<br />(3)如果调用 resolve 函数和 reject 函数时带有参数，那么它们的参数会被传递给回调函数。<br />(4)reject 函数的参数通常是 Error 对象的实例，表示抛出错误。<br />(5)resolve 函数的参数除了正常值外，还可以是另一个 Promise 实例。<br />上例中，p1 3秒后状态变为 rejected。p2 在 1秒后状态变为 resolved，resolve 方法返回 p1。由于 p2 返回的是另一个 Promise, 导致 p2 自己的状态无效了，由 p1 决定 p2 的状态，所以 then 方法变成针对 p1。<br />(6)调用 resolve 和 reject 并不会终结 Promise 的参数函数的执行。
```javascript
new Promise((resolve, reject) => {
	resolve(1);
	console.log(2);
}).then( r => {
	console.log(r)
})
// 2
// 1
```
上例先打印 2，再是1。是因为，立即 resolve 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。所以 resolve 或 reject 之后，Promise 的使命就完成了，后续操作应该放到 then 里面，不应该直接写在 resolve 或 reject 后面，最好在 resolve 或 reject 前面加上 return 。
<a name="AjFnD"></a>
#### 2. then 方法 
Promise 实例生成后，可以用 then 方法分别指定 resolved 状态和 rejected 状态的回调函数。<br />then 方法可以接受两个回调函数作为参数。第一个回调函数是 Promise 对象状态变为 resolved 时调用， 第二个回调函数是 Promise 状态变为 rejected 时调用。第二个函数是可选的，不一定要提供。这两个函数都接受 Promise 对象传去的值作为参数。
```javascript
function timeout(ms){
		return new Promise((reslove, reject) => {
				setTimeout(reslove, ms, 'done');//setTimeout 可以接受大于两个或更多的参数，从第三个参数开始，多余的参数都作为 resolve 函数的参数。
		})
}
timeout(100).then( value => {
		console.log(value);
		return '再次打印 done'
}).then(value => {
	console.log(value);
});
// done
// 再次打印 done
```

then 方法返回的是一个 新的 Promise 实例，因此可以采用链式写法，then 方法后面再调用另一个 then 方法。前一个 then 方法返回的结果会传入给后一个 then 方法。

<a name="J1pqH"></a>
#### 3. catch 方法 
catch 方法用于当 Promise 状态变为 rejected 时指定发生错误时的回调函数，相当于 then 方法的第二个参数函数。如果运行中抛出错误，也会被 catch 捕获。
```javascript
const doError = true;
const promise = new Promise((resolve, reject) => {
	if(doError){
		reject('操作失败');
	}
});
promise.catch((msg) => {
	console.log(msg);
});
// 操作失败

const promise = new Promise((resolve, reject) => {
  throw new Error('test');
});
promise.catch(error => {
  console.log(error);
});
// Error: test
```

(1)如果状态已经变为 resolved，再抛出错误，catch 不能捕获。因为 Promise 状态一旦改变不会再变。
```javascript
const promise = new Promise((resolve, reject) => {
  resolve('ok');
  throw new Error('test');
});
promise.then(
  value => { console.log(value) 
 }).catch(
  error => { console.log(error) 
 });
//ok
```

(2)Promise 对象的错误具有 “冒泡”性质，会一直向后传递，直到被捕获为止。即错误总是会被下一个 catch 捕获。<br />(3)最好使用 catch 捕获错误，而不是使用 then 的第二个参数。<br />(4)如果没有使用 catch 方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码。
```javascript

const promise =  new Promise((resolve, reject) => {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
});
promise.then(function() {
  console.log('everything is great');
});
setTimeout(() => { console.log(123) }, 2000);
// ReferenceError: x is not defined
// 123
```
上例中，会报错，但是不会终止脚本运行，2秒之后会输出 123，这就是 Promise内部的错误不会影响外部的代码。

(5)catch 方法返回的还是一个 Promise 对象，因此后面可以接着调用 then 方法。
<a name="knEe7"></a>
#### 4. finally 方法 
finally 方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。finally 方法的回调函数不接受任何参数。
```javascript
const promise = new Promise((resolve, reject) => {
	resolve(123);
});
promise.then((v) => {console.log(v)}).finally(() => {console.log(222)})
// 123
// 222
```
<a name="SQEnc"></a>
### 三、Promise 的方法
<a name="sVUuT"></a>
#### 1. Promise.all() 方法 
Promsie.all() 方法用于将多个 Promise 实例包装成一个新的 Promise 实例。
```javascript
const p = Promise.all([p1, p2, p3]);
```
Promsie.all() 的参数可以不是数组，但是必须有 Iterator 接口，且返回的每个成员都是 Promise。如果参数不是 Promise 实例，就会调用 Promise.resolve() 方法，将参数转为 Promise 实例，再处理<br />p 的状态由 p1、p2、p3 决定，分成两种情况：<br />(1) 只有 p1、p2、p3 的状态都是 fulfilled，p 的状态才是是 fulfilled，此时 p1、p2、p3 的返回值组成一个数组，传递给 P 的回调函数。<br />(2) 只要  p1、p2、p3  之中有一个被 rejected，p 的状态就变成 rejected，此时被 rejected 的实例的返回值，会传递给 p 的回调函数。

注意： 如果 p1、p2、p3 自己定义了 catch 方法，那么一旦被 rejected ，并不会触发 Promise.all() 的 catch 方法。如果没有自己的 catch 方法，就会调用 Promise.all() 的 catch 方法。
```javascript
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
}).then(result => result);
const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
}).then(result => result);

Promise.all([p1, p2]).then(result => console.log(result)).catch(e => console.log(e));
// Error: 报错了
```

<a name="TeJ9f"></a>
#### 2. Promise.race() 方法 
Promise.race() 方法也是将多个 Promise 实例，包装成一个新的 Promise 实例。参数与 Promise.all() 的一样。
```javascript
const p = Promise.race([p1, p2, p3]);
```
如果 p1、p2、p3 之中有一个实例的状态先改变， p 的状态就跟着变，最先改变状态的实例的返回值传递给 p 的回调函数。
<a name="JUWZr"></a>
#### 3. Promise.resolve() 方法 
Promsie.resolve() 将对象转为 Promise 对象。参数分为四种情况：<br />(1) 参数是一个 Promise 实例<br />那么 Promsie.resolve 将不做任何修改，原封不动的返回这个实例。<br />(2) 参数是一个 具有 then 方法的对象<br />那么  Promsie.resolve 会将这个对象转为 Promise 对象，然后立即执行对象的 then 方法。<br />(3) 参数不是具有 then 方法的对象，或者不是对象<br />那么  Promsie.resolve 会返回一个新的 Promise 对象，状态为 resolved。<br />(4) 不带任何参数<br />那么  Promsie.resolve 会返回一个新的 Promise 对象，状态为 resolved。<br />所以，如果想直接得到一个 Promise 对象，可以直接使用   Promsie.resolve() 方法。<br />注意： 立即 resolve() 的 Promise 对象，是在本轮 “事件循环”的结束时执行，而不是下一轮“事件循环”的开始执行。
```javascript
setTimeout(function () {
  console.log('three');
}, 0);
Promise.resolve().then(function () {
  console.log('two');
});
console.log('one');

// one
// two
// three
```
<a name="uZWfk"></a>
#### 4. Promise.reject() 方法 
Promise.reject() 方法返回一个 Promise 实例，实例的状态为 rejected。<br />注意：Promise.reject() 方法的参数，会原封不动的作为 reject 的理由，变为后续方法的参数。
```javascript
Promise.reject('出错了').catch(e => {
	console.log(e);
});
// 出错了
```

