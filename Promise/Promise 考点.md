
## 状态吸收

1. 基础知识

- `async` 函数会返回一个新的`Promise` **返回的 Promise 和 函数如果return的Promise 不是同一个**；
- `await` 后面可以不是`Promise`（通常是会一个async函数），但是不是的时候会自动被包装成一个`Promise`；
- `await` 的作用是等待后面的promise完成，然后将下面的代码放入到微队列中；
- `await` 只能出现在`async`标记的函数内部；
- `async` 函数可以直接调用；
```js
const p1 = Promise.resolve();

async function async1(){
	return p1
};

const p2 = async1();

console.log(p1 === p2);   // false
```


2. 常见和常考的的吸收情况
- 案例1
```js
const p1 = new Promise((resolve) => {
	resolve(100)
})
const p2 = new Promise((resolve) => {
	resolve(p1)   // resolve一个promise
})

p2.then(() => {
	console.log('1')
})
	.then(() => {
		console.log('2')
	})
	.then(() => {
		console.log('3')
	})

p1.then(() => {
	console.log('4')
})
	.then(() => {
		console.log('5')
	})
	.then(() => {
		console.log('6')
	})
	
// 控制台输出
// 4 5 1 6 2 3	
```
*or*
- 案例2
```js
const p1 = Promise.resolve();

async function async1(){
	return p1
};

const p2 = async1();

console.log(p1 === p2);   // false

// p2 要吸收 p1 的状态
```
*or*
- 案例3
```js
const p1 = Promise.resolve()
const p2 = Promise.resolve(p1) 
```
*or*
- 案例4
```js
const p1 = Promise.resolve()
const p2 = Promise.resolve().then(()=>p1)
```

上面是常见的几种 **==状态吸收==** 的情况（p2的状态由p1的状态来决定）

- 第三种情况就是`Promise`源码内置的`then`方法中的吸收情况和逻辑！
- 吸收过程分为两步
	- 1. **准备**（就是将吸收的逻辑代码被一个函数包裹起来，并将这个方法放入到==**微队列中**==等待执行）
	- 2. ==**吸收**（这一步也是放入到**微队列里**等待执行；原理就是，将p2的`resolve` 和 `reject` 注册成 p1.then 中的回调函数`onFullfilled` 和 `onRejected` ；从而实现了p1 resolve之后 p2也会自动 resolve ）==


>实际==案例1 ==的 执行逻辑
```js
//状态吸收（p2 吸收 p1）： 准备  吸收  --- p1完成，p2挂起（准备吸收状态）
微队列： 准备 4
控制台： 



//状态吸收： 准备  吸收     --- `准备`已执行完成，将`吸收`放入微队列
微队列： 4 吸收
控制台： 


//状态吸收： 准备  吸收     --- `准备`已执行完成，将`吸收`放入微队列     
微队列： 吸收 5
控制台： 4


//状态吸收： 准备  吸收     --- `吸收`已执行完成，p2完成， 将`1`放入微队列
微队列： 5  1
控制台： 4 


//状态吸收： 准备  吸收     --- 输出 `5`，将`6`放入微队列
微队列： 1 6
控制台： 4 5


//状态吸收： 准备  吸收     --- 输出 `1`，将`2`放入微队列
微队列： 6 2
控制台： 4 5 1 


//状态吸收： 准备  吸收     --- 输出 `6`和`2`，将`3`放入微队列
微队列： 3
控制台： 4 5 1 6 2

//状态吸收： 准备  吸收     --- 输出`3`
微队列： 
控制台： 4 5 1 6 2 3
```