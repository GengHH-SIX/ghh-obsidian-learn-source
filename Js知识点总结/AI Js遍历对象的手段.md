问：给我详细对比一下js中常用的遍历手段。比如 for，for...in，for..of， forEach等。需要能涉及他们触发元属性的知识，比如 for..in 会触发 in 符号来操作对象，for..of 是 和Symbol.Iterator 等等；

答：
基于历史对话背景（涉及 `Proxy` 拦截、`has()` 陷阱、`Object.hasOwn` 与 `in` 的区别），在对比遍历手段时，必须重点分析它们如何与 Proxy 交互以及是否触发元编程陷阱（Traps）。

以下是 JavaScript 常用遍历手段的深度对比，特别关注底层机制与 Proxy 拦截行为：

## 一、核心遍历手段对比总表

| 遍历方式          | ==适用==对象         | ==底层==机制/协议       | 是否触发 Proxy ==has()== | 是否触发 Proxy ==get()== | 是否遍历==原型链== | 可==中断== (break) |
| ------------- | ---------------- | ----------------- | -------------------- | -------------------- | ----------- | --------------- |
| ‌for...in‌    | 对象 (Object)      | 枚举键 (Enumeration) | ✅ ‌是‌                | ❌ 否 (仅获取键)           | ✅ ‌是‌       | ❌ 否 (需 return)  |
| ‌for...of‌    | 可迭代对象 (Iterable) | Symbol.iterator   | ❌ 否                  | ✅ ‌是‌ (获取值)          | ❌ 否         | ✅ ‌是‌           |
| ‌forEach‌     | 数组 (Array)       | 内部  ==Get==+ 索引循环 | ❌ 否                  | ✅ ‌是‌                | ❌ 否         | ❌ 否 (需抛异常)      |
| ‌for (传统)‌    | 数组/类数组           | 索引访问 obj[i]       | ❌ 否                  | ✅ ‌是‌                | ❌ 否         | ✅ ‌是‌           |
| ‌Object.keys‌ | 对象 (Object)      | 枚举==自身==可枚举键      | ❌ 否                  | ❌ 否                  | ❌ 否         | N/A             |
  
or

| 特性 | for await...of | for...of | forEach | for (传统) | for...in |
| --- | --- | --- | --- | --- | --- |
| ‌适用对象‌ | ‌异步迭代器‌ (Async Iterable) | 同步迭代器 (Iterable) | 数组 (Array) | 数组/类数组/对象 | 对象 (Object) |
| ‌异步支持‌ | ✅ ‌原生支持‌ (自动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 |
| ‌底层协议‌ | Symbol.asyncIterator | Symbol.iterator | 内部索引循环 | 索引访问 | 枚举键 |
| ‌Proxy 拦截‌ | 触发 get(Symbol.asyncIterator) | 触发 get(Symbol.iterator) | 触发 get('length'), get(index) | 触发 get(index) | 触发 has()/枚举 |
| ‌可中断‌ | ✅ ‌支持‌ (break/return) | ✅ 支持 | ❌ 不支持 | ✅ 支持 | ❌ 不支持 |
| ‌执行顺序‌ | ‌串行‌ (等待前一个 resolve) | 同步串行 | 同步串行 | 同步串行 | 同步串行 |
---


## 二、详细机制与 Proxy 拦截分析

#### 0. `for await...of`
`for await...of` 是 ES2018 引入的专门用于遍历‌**异步迭代器（Async Iterators）**‌的语法。它与之前提到的 `for`、`for...in`、`for...of`、`forEach` 有本质区别，核心在于处理 ‌**Promise**‌ 和‌**异步数据流**‌。

==**`详情查看文档末尾（异步遍历）知识点`**==


#### 1. `for...in`：基于枚举与 `has` 陷阱

- 底层逻辑：

  - 遍历对象及其原型链上所有可枚举（enumerable）的字符串键。
  - 关键行为：在检查属性是否存在于对象或原型链时，理论上会涉及属性存在性检查。

- Proxy 交互：

  - 触发 `has()` 陷阱：虽然规范主要强调枚举，但在某些实现或特定场景下，判断属性可见性可能间接关联到存在性检查。更重要的是，它不触发 `get()`，只获取键名。
  - 注意：`for...in` ==不会遍历 `Symbol` 键==，也不会遍历不可枚举属性（如 `defineProperty` 设置的 `enumerable: false`）。

  **注意：** 遍历稀松数组\[1,undefined,2,\]时候,会跳过空的属性值，==不会访问空槽==
		通常会和Object.hasOwn() 组合使用，来访问对象自身的属性
  
- 代码示例：
  ```javascript
  const proxy = new Proxy({ a: 1 }, {
    has(target, key) {
      console.log(`has trapped: ${key}`);
      return true; // 强制认为所有属性都存在
    }
  });

  for (const key in proxy) {
    console.log(key); // 'a'
    // 注意：has 陷阱可能在内部枚举过程中被调用，具体取决于引擎实现，但主要风险在于它遍历原型链。
  }
  ```

#### 2. `for...of`：基于迭代协议 `Symbol.iterator`

- 底层逻辑：

  - 调用对象的 `[Symbol.iterator]()` 方法获取迭代器对象。
  - 不断调用迭代器的 `.next()` 方法，直到 `done: true`。

- Proxy 交互：

  - 触发 `get()` 陷阱：为了获取 `Symbol.iterator` 方法以及每次获取 `.next()` 返回的 `value`，会多次触发 `get` 陷阱。
  - 不触发 `has()`：它不关心属性是否存在，只关心迭代协议。
  - 不遍历原型链属性：除非原型链上的对象实现了迭代器且被代理覆盖，否则它只处理当前对象的迭代逻辑。

- 代码示例：
  ```javascript
  const arr = [1, 2, 3];
  const proxy = new Proxy(arr, {
    get(target, prop, receiver) {
      console.log(`get trapped: ${String(prop)}`);
      return Reflect.get(target, prop, receiver);
    }
  });

  for (const val of proxy) {
    console.log(val); 
    // 输出: get trapped: Symbol(Symbol.iterator)
    //       get trapped: 0, 1, 2 (获取值时触发)
  }

  ```
  
  **注意：** 遍历稀松数组\[1,undefined,2,\]时候,会跳过空的属性值，==会访问空槽==
#### 3. `forEach` / `map` / `filter`：基于索引的内部循环

- 底层逻辑：

  - 数组内部维护一个长度 `length`。
  - 从索引 `0` 到 `length - 1`，通过内部 `[[Get]]` 操作获取值并执行回调。

- Proxy 交互：

  - 触发 `get()` 陷阱：每次访问 `arr[index]` 时触发。
  - 触发 `get('length')`：首先获取长度以确定循环次数。
  - 不触发 `has()`：数组方法通常假设索引存在性由 `length` 决定，不进行显式的 `in` 检查。

- 缺陷：无法使用 `break` 或 `continue`，只能通过抛出异常中断（不推荐）。

  
#### 4. 传统 `for` 循环：最底层的索引访问

- 底层逻辑：

  - 手动维护索引变量 `i`。
  - 直接通过 `obj[i]` 访问属性。

- Proxy 交互：

  - 触发 `get()` 陷阱：每次 `obj[i]` 读取时触发。
  - 完全可控：你可以精确控制何时停止，避免不必要的 Proxy 调用。

- 优势：性能最高，无额外函数调用开销，对 Proxy 的干扰最小（仅必要的 `get`）。


---

  
## 三、特殊情形：`Object.keys` / `values` / `entries`
  
- 机制：返回对象自身可枚举属性的数组。

- Proxy 交互：
  - 触发 `ownKeys()` 陷阱：这是关键！`Object.keys` 会触发 Proxy 的 `ownKeys` 陷阱，而不是 `has` 或 `get`。
  - 不遍历原型链：仅返回自身属性。

- 代码示例：
  ```javascript
  const proxy = new Proxy({ a: 1 }, {
    ownKeys(target) {
      console.log('ownKeys trapped');
      return ['a', 'b']; // 可以伪造键
    },
    getOwnPropertyDescriptor(target, prop) {
      // Object.keys 还需要检查 enumerable，因此也可能触发此陷阱
      return { enumerable: true, configurable: true };
    }
  });

  console.log(Object.keys(proxy)); // ['a', 'b']
  ```

##  四、总结与建议
####  1. 若需拦截“属性存在性检查”：
   - 使用 `for...in` 或 `in` 操作符。它们与 `has()` 陷阱紧密相关。
   - 注意：`Object.hasOwn()` 不会触发 `has()` 陷阱，它是安全的底层检查。

#### 2. 若需拦截“值访问”：
   - 使用 `for...of`、`forEach` 或 `for`。它们都依赖 `get()` 陷阱。
   - `for...of` 还会触发 `Symbol.iterator` 的 `get`。

#### 3. 若需拦截“键枚举”：
   - 使用 `Object.keys()` 或 `for...in`。
   - `Object.keys` 触发 `ownKeys()`，更纯粹地反映自身结构。
   - `for...in` 混合了原型链，且在 Proxy 中行为较复杂（可能受 `has` 影响）。
#### 4. 最佳实践：
   - 遍历数组：优先用 `for...of`（可读性好）或 `for`（性能好/可控）。
   - 遍历对象：优先用 `Object.keys/entries` + `for...of`，避免 `for...in` 的原型链污染问题。
   - Proxy 开发：明确区分 `ownKeys`（枚举）、`has`（存在性）、`get`（取值）三个陷阱，根据遍历方式选择正确的拦截点。
   
   参考资料<br>[1] [【2025前端】JavaScript面试合集,前端面试题之对象遍历方式深度对比。2025最新大厂前端面试真题讲解,看完在去面试! - 老油条说面试](http://www.bilibili.com/video/BV1Nn8RzpEBD)<br>[2] [js的15种循环遍历,你掌握了几种? - 腾讯云](https://cloud.tencent.com/developer/article/1641899)<br>[3] [JS几种数组遍历方式总结 - 腾讯云](https://cloud.tencent.com/developer/article/2106200)<br>[4] [JS数组遍历的几种方法 - 腾讯云](https://cloud.tencent.com/developer/article/2105296)<br>[5] [【前端面试精选】JS每日一问:对象遍历方式对比(for in、Object.keys、Reflect) - 土豆的笔记](http://www.bilibili.com/video/BV15rzNBMEnM)<br>[6] [JS遍历循环方法性能对比:for/while/for in/for of/map/foreach/every - 腾讯云](https://cloud.tencent.com/developer/article/1867339)<br>[7] [JS遍历循环的5种方法性能对比:for , foreach , map ,for...in , for...of - 前端学习站](https://zhuanlan.zhihu.com/p/521916572)<br>[8] [前端数组遍历该用什么方法?带你一探究竟! - DEMO娲](http://zhuanlan.zhihu.com/p/1989298419085382348)<br>[9] [javascript 遍历元素属性。 - CSDN博客](https://blog.csdn.net/weixin_34123613/article/details/91829922)<br>[10] [JavaScript中常用的的遍历方法--总结 - CSDN博客](https://blog.csdn.net/x15514104477/article/details/140084798)<br>[11] [in操作符怎么配合for...in循环使用?怎么规避性能隐患? - php中文网](https://www.php.cn/faq/2635092.html)<br>[12] [如何规避在for-in循环中使用in操作符引发的原型链属性污染遍历漏洞 - php中文网](https://www.php.cn/faq/2693847.html)<br>[13] [for ... in ... 的实现原理(纯干货) - CSDN博客](https://blog.csdn.net/qq_39618672/article/details/145859001)<br>[14] [JavaScript的第七种数据类型,Symbol的作用,Symbol.iterator与for...of - 晓舟报告](http://www.bilibili.com/video/BV1vo4y1D7kD)<br>[15] [JavaScript对象属性遍历方法总结 - 笑傲存钱罐](http://quanmin.baidu.com/sv?source=share-h5&pd=qm_share_search&vid=14590660232375187657)<br>[16] [JavaScript数组遍历的两种方式 - marry的线上教育](http://quanmin.baidu.com/sv?source=share-h5&pd=qm_share_search&vid=5705692942046639574)<br>[17] [告别复杂,轻松掌握JavaScript循环遍历! - 车侠说驾](http://haokan.baidu.com/v?pd=wisenatural&vid=16446358308095804468)<br>[18] [【JavaScript前端教程】JavaScript2小时快速入门,全程无废话,入门到精通,前端JS全套基础/实战教程,附源码+文档 - 清月WOW](http://www.bilibili.com/video/BV19stCzAECZ?p=7)<br>[19] [JavaScript循环全解析:从for到for...of,掌握高效遍历与避坑指南 - CSDN博客](https://blog.csdn.net/weixin_26763955/article/details/163709604)<br>[20] [JS基础以及与Java区别学习笔记 - 你可曾有我 - 博客园 - 博客园](https://www.cnblogs.com/lie-explore-lie/p/22321834)<br>[21] [JavaScript 学习-4.Array数组遍历的几种方式 - 腾讯云](https://cloud.tencent.com/developer/article/2003999)<br>[22] [JS数组和对象的遍历方式,以及几种方式的比较 - 腾讯云](https://cloud.tencent.com/developer/article/2372189)<br>[23] [Javascript 10种遍历方法 - 乔二哥 - 博客园 - 博客园](https://www.cnblogs.com/qiaoge0923/p/9594539.html)<br>[24] [JavaScript 对象属性遍历方法全解析:深度对比与最佳实践 - 博客园](https://www.cnblogs.com/hesw/articles/18806148)<br>[25] [吃透JS 遍历:从基础到进阶的全维度解析__js 各遍历方法的区别-CSDN博客 - CSDN博客](_https://blog.csdn.net/m0_73869974/article/details/156910170_)_<br>_[26] [JS 中的 24 种遍历方法全解析:从基础到进阶 - CSDN博客](_https://blog.csdn.net/m0_73869974/article/details/154767343_)_<br>_[27] [JavaScript数组遍历方法全面解析forEach、map、filter与reduce的深度对比 - CSDN博客](_https://blog.csdn.net/tDUARHTn/article/details/154791448_)_<br>_[28] [JavaScript遍历方法全解析:for、for...in、for...of、forEach、map、reduce手写实战-CSDN博客 - CSDN博客](_https://blog.csdn.net/weixin_36074800/article/details/154454261_)_<br>_[29] [24种 JavaScript 循环遍历方法 - CSDN技术社区](_https://gongjianbo1992.blog.csdn.net/article/details/128438562_)_<br>_[30] [JavaScript数组遍历方法全解析 - 小程玩编程](_http://mbd.baidu.com/newspage/data/dtlandingsuper?nid=dt_5344800876147177951_)_<br>_[31] [JavaScript对象遍历方法详解 - 小程玩编程](_http://mbd.baidu.com/newspage/data/dtlandingsuper?nid=dt_4290423693950079495_)_<br><br>_百度AI生成，内容仅供参考_


--------

`for await...of` 是 ES2018 引入的专门用于遍历异步迭代器（Async Iterators）的语法。它与之前提到的 `for`、`for...in`、`for...of`、`forEach` 有本质区别，核心在于处理 Promise 和异步数据流。

### 一、核心对比总表
| 特性 | for await...of | for...of | forEach | for (传统) | for...in |
| --- | --- | --- | --- | --- | --- |
| ‌适用对象‌ | ‌异步迭代器‌ (Async Iterable) | 同步迭代器 (Iterable) | 数组 (Array) | 数组/类数组/对象 | 对象 (Object) |
| ‌异步支持‌ | ✅ ‌原生支持‌ (自动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 (需手动 await) | ❌ 不支持 |
| ‌底层协议‌ | Symbol.asyncIterator | Symbol.iterator | 内部索引循环 | 索引访问 | 枚举键 |
| ‌Proxy 拦截‌ | 触发 get(Symbol.asyncIterator) | 触发 get(Symbol.iterator) | 触发 get('length'), get(index) | 触发 get(index) | 触发 has()/枚举 |
| ‌可中断‌ | ✅ ‌支持‌ (break/return) | ✅ 支持 | ❌ 不支持 | ✅ 支持 | ❌ 不支持 |
| ‌执行顺序‌ | ‌串行‌ (等待前一个 resolve) | 同步串行 | 同步串行 | 同步串行 | 同步串行 |
### 二、深度机制分析

#### 1. `for await...of`：异步迭代协议

- 底层逻辑：
  - 调用对象的 `[Symbol.asyncIterator]()` 方法获取异步迭代器。
  - 每次调用 `.next()` 返回一个 Promise。
  - 自动 `await` 该 Promise，直到 resolve 后获取 `value`，再进入下一次循环。

- Proxy 交互：
  - 触发 `get()` 陷阱：获取 `Symbol.asyncIterator` 属性时触发。
  - 不触发 `has()`：与 `for...of` 类似，不关心属性存在性，只关心迭代协议。

- 适用场景：
  - 读取大型文件流（Node.js `fs.createReadStream`）。
  - 分页获取 API 数据（每次请求依赖上一次的结果）。
  - 处理 WebSocket 消息流。

```javascript
// 示例：异步生成器
async function* asyncGenerator() {
  yield await fetch('/api/page1');
  yield await fetch('/api/page2');
}
  
// ✅ 正确：串行执行，等待每个 Promise 完成
for await (const data of asyncGenerator()) {
  console.log(data); 
}
```

#### 2. `for...of` vs `for await...of`

- 关键区别：

  - `for...of` 期望 `.next()` 返回 `{ value, done }` 普通对象。
  - `for await...of` 期望 `.next()` 返回 `Promise<{ value, done }>`。

- 错误用法：
  ```javascript
  // ❌ 错误：for...of 不会自动 await，data 是 Promise 对象
  for (const data of asyncGenerator()) {
    console.log(data); // Promise { <pending> }
  }
  ```

#### 3. `forEach` 与异步的陷阱

- 致命缺陷：`forEach` 不等待 异步操作完成。
- 后果：循环会瞬间结束，所有异步任务并发启动，且无法捕获错误或使用 `break`。

- 对比：
  ```javascript
  // ❌ 危险：并发执行，无法控制顺序，无法 break
  [1, 2, 3].forEach(async (item) => {
    await fetch(`/api/${item}`);
  });
  console.log('Done'); // 立即输出，此时请求可能还没发出去
  ```


#### 4. 传统 `for` 循环与异步

- 可行性：可以在传统 `for` 中使用 `await`，效果与 `for await...of` 类似（串行执行）。
- 缺点：代码冗长，需手动管理索引和迭代器逻辑。
- 优点：兼容性最好，无需对象实现迭代协议。
```javascript
// ✅ 可行：串行执行，但代码啰嗦
const items = [1, 2, 3];
for (let i = 0; i < items.length; i++) {
  await fetch(`/api/${items[i]}`);
}
```

### 三、Proxy 拦截行为对比

若对象被 Proxy 代理，不同循环触发的陷阱不同：

| 循环类型 | 触发的 Proxy Trap | 说明 |
| --- | --- | --- |
| ‌for await...of‌ | get(target, Symbol.asyncIterator) | 获取异步迭代器方法 |
| ‌for...of‌ | get(target, Symbol.iterator) | 获取同步迭代器方法 |
| ‌forEach‌ | get(target, 'length'), get(target, index) | 获取长度和每个元素 |
| ‌for...in‌ | ownKeys(), getOwnPropertyDescriptor(), has() | 枚举键并检查存在性 |

注意：如果 Proxy 只实现了 `Symbol.iterator` 而未实现 `Symbol.asyncIterator`，使用 `for await...of` 会报错（除非该同步迭代器返回的值本身就是 Promise，但这不符合规范最佳实践）。

### 四、总结与建议

#### 1. 处理异步数据流：

   - 首选：`for await...of`。语法简洁，天然支持串行等待和 `break`。
   - 前提：数据源必须实现 `Symbol.asyncIterator`（如 Node.js Streams、自定义异步生成器）。

#### 2. 处理普通数组/对象：

   - 首选：`for...of`（数组/Map/Set）或 `Object.keys()` + `for...of`（对象）。
   - 避免：`forEach` 处理异步逻辑。
  
#### 3. 性能与控制：

   - ==若需要并发执行异步任务（而非串行），应使用 `Promise.all()` 配合 `map`，而不是 `for await...of`。==
     ```javascript
     // 并发请求：比 for await...of 快，因为同时发起
     await Promise.all(urls.map(url => fetch(url)));
     ```

#### 4. Proxy 开发提示：

   - 若需代理异步可迭代对象，务必在 Proxy 的 `get` 陷阱中正确处理 `Symbol.asyncIterator`，返回一个符合规范的异步迭代器对象。


   参考资料<br>[1] [JS 中的流式处理技术 Iterator、Generator、for await...of 【通俗易懂 AI 应用开发】 - 捣鼓键盘的小麦](http://www.bilibili.com/video/BV1tUagzGEJ4)<br>[2] [JavaScript 异步遍历器实战指南 for await of - 开源中国](https://my.oschina.net/emacs_8005672/blog/19692570)<br>[3] [JavaScript|第27章:生成器与异步迭代 - CSDN博客](https://blog.csdn.net/xhmico/article/details/163139973)<br>[4] [for await...of 深度解析:从协议、return() 到 finally 的完整链路 - www.nicesgo.cn](http://www.nicesgo.cn/tech-notes/for-await-of-deep-dive.html)<br>[5] [从底层看前端(十二)—— JavaScript语法:JavaScript有哪些语句? - 陈大饼](http://zhuanlan.zhihu.com/p/143958679)<br>[6] [Proxy中怎么利用元编程能力将一个普通对象包装成可以被异步迭代(for await...of)的流数据 - php中文网](https://www.php.cn/faq/2875300.html)<br>[7] [Proxy、reflect、Iterator、Promise、Generator函数、async - CSDN博客](https://blog.csdn.net/chenjia525/article/details/125529457)<br>[8] [QuickJS高级特性探索:BigInt、异步生成器和Proxy的完整指南 - CSDN博客](https://blog.csdn.net/gitblog_00929/article/details/155875440)<br>[9] [一次性搞明白 5 种 for 循环的用法 - 腾讯云](https://cloud.tencent.com/developer/article/1983794)<br>[10] [遍历数组的四种方法对比:for vs for-in vs forEach() vs for-of(译文) - DJ荒野 - 博客园 - 博客园](https://www.cnblogs.com/reamd/articles/14944972.html)<br>[11] [js ES6 Iterator 遍历器与 for、for...of、for await...of、for...in 和 forEach 循环语句__es6 遍历执行 指针-CSDN博客 - CSDN博客](_https://blog.csdn.net/mChales_Liu/article/details/107188470_)_<br>_[12] [JS中 for...in 和 for...of 的区别 - 陈鹤鸣2333](_http://zhuanlan.zhihu.com/p/694238480_)_<br>_[13] [for await...of 和for....of的区别是什么 - CSDN博客](_https://blog.csdn.net/qq_20173195/article/details/134057021_)_<br>_[14] [JavaScript中的for...of和for await...of循环 - CSDN博客](_https://blog.csdn.net/debug_bug142/article/details/133158757_)_<br>_[15] [为什么你的Lint工具总在提示用 for...of 替代 forEach? - 阿张](_https://zhuanlan.zhihu.com/p/1908164389577668101_)_<br>_[16] [【前端面试精选】JS每日一问:async、await 的底层运行机制 - 土豆的笔记](_http://www.bilibili.com/video/BV1ReFhz8EVP_)_<br>_[17] [JavaScript 运行机制详解——高级前端专家必懂的 JS 底层知识 - 前端工程化](_http://www.bilibili.com/video/BV1VcGRzsEdw?p=6_)_<br>_[18] [JavaScript 运行机制详解——高级前端专家必懂的 JS 底层知识 - 前端工程化](_http://www.bilibili.com/video/BV1VcGRzsEdw?p=7_)_<br>_[19] [async/await 原理深度解析:JavaScript异步编程底层工作机制全揭秘 - 腾讯云](_https://cloud.tencent.com/developer/news/2549966_)_<br>_[20] [JavaScript 异步编程指南 — 终极解决方案 Async/Await - 腾讯云](_https://cloud.tencent.com/developer/article/1847713_)_<br>_[21] [JavaScript循环全解析:从for到for...of,掌握高效遍历与避坑指南 - CSDN博客](_https://blog.csdn.net/weixin_26763955/article/details/163709604_)_<br>_[22] [深入解析JavaScript事件循环机制与异步编程 - CSDN博客](_https://blog.csdn.net/weixin_32642109/article/details/163691210_)_<br>_[23] [for-await-of 底层实现原理详解 - CSDN博客](_https://blog.csdn.net/a181001_/article/details/148022125_)_<br>_[24] [深入解析 for await...of:异步迭代的强大工具 - CSDN博客](_https://blog.csdn.net/gusushantang/article/details/141167714_)_<br>_[25] [浅析异步循环for await of的使用及执行机制及for of/forEach本质区别和遇到异步时的处理 - 博客园](_https://www.cnblogs.com/goloving/p/16009741.html_)_<br>_[26] [JavaScript 异步编程终极语法(async/await ) - CSDN博客](_https://blog.csdn.net/wyp20151314/article/details/161033145_)_<br>_[27] [async/await 深度解析 - CSDN博客](_https://blog.csdn.net/qq_39302722/article/details/162207793_)_<br>_[28] [JavaScript 异步迭代:for-await-of 循环处理异步数据流 - php中文网](_https://www.php.cn/faq/1668419.html_)_<br>_[29] [JS循环中调用异步函数,为什么forEach中async、await无效,及解决方法(for循环的使用和终止,Promise.all并行,for await...of 串行)__js foreach中await无效-CSDN博客 - CSDN博客](https://blog.csdn.net/zyhui994/article/details/155033950)<br><br>百度AI生成，内容仅供参考