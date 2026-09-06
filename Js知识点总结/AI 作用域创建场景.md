
在 JavaScript（特别是 ES6+）中，普通函数并非创建作用域的唯一方式，且块级作用域的引入彻底改变了这一认知。

注意：正常情况下，在任何位置访问某个变量，都会沿着当前作用域向上层作用域中查找
### 一、JavaScript 中的作用域类型

现代 JavaScript 主要包含以下==**三种**==作用域，其中两种不需要函数即可创建：

#### 1. 全局作用域 (Global Scope)

- 创建方式：脚本最外层。
- 特点：在整个程序中可访问。

- 示例：
  ```javascript
  var globalVar = 'I am global'; // 全局作用域，会变量提升 
  let globalVar = 'I am global'; // 全局作用域，形成暂时性死区，不会变量提升
  const globalVar = 'I am global'; // 全局作用域，形成暂时性死区，不会变量提升
  ```

#### 2. 函数作用域 (Function Scope)

- 创建方式：普通函数、箭头函数、构造函数等。
- 特点：只有进入函数内部才能访问变量。这是 ES5 及以前唯一能创建局部作用域的方式。

- 示例：
  ```javascript
  function myFunc() {
    var funcVar = 'I am local'; // 函数作用域，只在此方法中变量提升，不回跑到上层作用域中
    // let 只在此方法中，形成暂时性死区，不会变量提升
    // const 只在此方法中，形成暂时性死区，不会变量提升
  }
  ```

#### 3. 块级作用域 (Block Scope) —— 关键区别

- 创建方式：==**使用 `{}` 代码块，配合 `let` 或 `const` 声明**==。
- 特点：不需要函数即可创建独立作用域。这是 ES6 引入的重大特性。

- 示例：
  ```javascript
	if (true) {
		let blockVar = 'I am block scoped'; // 块级作用域 + let
		console.log(blockVar); // 'I am block scoped'
	}
	// console.log(blockVar); // ❌ ReferenceError: blockVar is not defined
   
    //other
  
	if (true) {
	  var blockVar = 'I am block scoped';//块级作用域+var（或者说是没有形成块级作用域名）
	  console.log(blockVar); // 'I am block scoped'
	}
	console.log(blockVar); // 'I am block scoped'
	// 会输出：
	//I am block scoped
	//I am block scoped。 //var blockVar 会从块级作用域中提升
  
  ```


---
  
### 二、常见误区澄清

#### 1. `var` 不创建块级作用域

- `var` 声明的变量只受函数作用域和全局作用域限制，忽略块级 `{}`。

  ```javascript
  if (true) {
    var oldVar = 'I leak out'; 
  }
  console.log(oldVar); // 'I leak out' (变量泄露到外部)

  ```

#### 2. 其他创建作用域的结构

除了普通函数和块，以下结构也会创建块级作用域（针对 `let/const`）：

- ==`for` / `while` 循环：==
  ```javascript
  for (let i = 0; i < 5; i++) {
    // i 仅在循环块内有效
  }
  // console.log(i); // ❌ ReferenceError
  ```


- ==`try...catch`：==
  ```javascript
  try {
    // ...
  } catch (err) {
    // err 仅在 catch 块内有效
  }
  ```

#### 3. 模块作用域 (Module Scope)

- 创建方式：ES Module (`import/export`)。
- 特点：每个 `.js` 文件若作为模块加载，其顶层作用域是独立的，不会污染全局对象 `window`。

  ```javascript
  // module.js
  const moduleVar = 'I am private to this module';
  // 其他文件无法直接访问 moduleVar，除非通过 export 导出
  ```

---


### 三、总结对比

| 作用域类型   | 创建条件           | 关键字支持                     | 是否需函数 |
| ------- | -------------- | ------------------------- | ----- |
| ‌全局作用域‌ | 脚本顶层           | var, let, const, function | ❌ 否   |
| ‌函数作用域‌ | 函数内部           | var, let, const, function | ✅ 是   |
| ‌块级作用域‌ | {} 代码块内        | ‌仅 let, const‌            | ❌ ‌否‌ |
| ‌模块作用域‌ | ES Module 文件顶层 | import, export            | ❌ 否   |

### 四、结论

- 错误观点：“只有普通函数能创建作用域”。

- 正确观点：
  1. 函数创建函数作用域。
  2. 代码块 `{}` 配合 `let/const` 创建块级作用域（无需函数）。
  3. 模块文件创建模块作用域。
  4. 全局本身就是最外层作用域。
  
**在现代开发中，应优先使用 `let` 和 `const` 利用块级作用域来避免变量泄露，而不是依赖函数来隔离变量**。<br>参考资料<br>[1] [js进阶教程 - 筱睆](http://www.bilibili.com/video/BV1gN2KYoEb3?p=120)<br>[2] [js函数篇,函数和函数表达式,函数作用域,全局作用域 - CSDN博客](https://blog.csdn.net/F_fengzilin/article/details/116296110)<br>[3] [Javascript中的作用域 - 知了好学](https://xue.baidu.com/okam/pages/strategy/index?strategyId=121207600566728&source=natural)<br>[4] [图解javascript作用域 · Issue #39 · AttemptWeb/Record · GitHub - GitHub](https://github.com/AttemptWeb/Record/issues/39)<br>[5] [深入JavaScript作用域:词法解析与块级作用域实战指南 - 百度智能云](https://cloud.baidu.com/article/4626792)<br>[6] [深入解析JavaScript:作用域与作用域链的完整指南 - 百度智能云](https://cloud.baidu.com/article/4599024)<br>[7] [JavaScript-词法作用域 - 李立超老师](http://www.bilibili.com/video/BV1zw4m1k78z)<br>[8] [JavaScript 中的作用域分类 - 腾讯云](https://cloud.tencent.com/developer/article/2689338)<br>[9] [JS 的 9 种作用域,你能说出几种? - 腾讯云](https://cloud.tencent.com/developer/article/2212250)<br>[10] [JavaScript-作用域、块级作用域、上下文、执行上下文、作用域链 - 腾讯云](https://cloud.tencent.com/developer/article/1402536)<br>[11] [深入JavaScript闭包(二):作用域,作用域链 - 烈风逍遥](https://zhuanlan.zhihu.com/p/644695362)<br>[12] [JavaScript 函数与作用域深度详解:从基础语法到底层机制(新手必懂)-CSDN博客 - CSDN博客](https://blog.csdn.net/2602_95057108/article/details/162955416)<br>[13] [【JavaScript】作用域 ① ( JavaScript 作用域 | 全局作用域 | 局部作用域 | JavaScript 变量 | 全局变量 | 局部变量 ) - CSDN技术社区](https://hanshuliang.blog.csdn.net/article/details/137400212)<br>[14] [深度解析JS作用域机制:从基础到作用域链实践 - 百度开发者中心](https://developer.baidu.com/article/detail.html?id=4626936)<br>[15] [JavaScript 中的“作用域”是什么意思? - 腾讯云](https://cloud.tencent.com/developer/article/2355898)<br>[16] [全局作用域、函数作用域 - CSDN下载](https://download.csdn.net/blog/column/10358531/121530612)<br>[17] [JS---进阶 - CSDN博客](https://blog.csdn.net/2501_93898031/article/details/157842847)<br>[18] [javascript学习中自己对作用域和作用域链理解 - 博客园](https://www.cnblogs.com/lsy0403/p/5847276.html)<br>[19] [JS作用域 - CSDN博客](https://blog.csdn.net/qq_26942049/article/details/126295287)<br>[20] [JavaScript基础(三)函数、作用域 - CSDN博客](https://blog.csdn.net/weixin_53072519/article/details/119008087)<br>[21] [登录 - CSDN博客](https://blog.csdn.net/luckyxinxinziYa/article/details/134168730)<br>[22] [javascript中的作用域 - CSDN博客](https://blog.csdn.net/weixin_50161578/article/details/116656032)<br>[23] [JavaScript 作用域 - CSDN博客](https://blog.csdn.net/weixin_44960636/article/details/132344157)<br>[24] [js的作用域有哪些? - CSDN博客](https://blog.csdn.net/2401_87873725/article/details/147158265)<br>[25] [JavaScript执行机制:变量提升、作用域链、词法作用域、块级作用域、闭包和this - 腾讯云](https://cloud.tencent.com/developer/article/2438301)<br>[26] [【学习笔记】JS基础-作用域和闭包 - CSDN博客](https://blog.csdn.net/nameofacity/article/details/144864377)<br>[27] [🌐 JavaScript作用域全解析 - 世界知识望远镜](https://mbd.baidu.com/newspage/data/dtlandingsuper?nid=dt_5430446269484376577)<br>[28] [🤔终于搞懂作用域啦!🎉 - 洛城的桑田一粒沙](http://mbd.baidu.com/newspage/data/dtlandingsuper?nid=dt_5379456726838645123)<br>[29] [JavaScript作用域 - 智步DataV](http://www.bilibili.com/video/BV11nd3YWE1Q)<br>[30] [JavaScript变量作用域与机制解析 - 清风无痕](http://quanmin.baidu.com/sv?source=share-h5&pd=qm_share_search&vid=12147994387386436314)<br>[31] [JS中作用域问题 - TodoCode前端](http://haokan.baidu.com/v?pd=wisenatural&vid=5561978618951646246)<br>[32] [【2024最新版】JavaScript零基础入门到精通(已完结) - 橙序进行中](http://www.bilibili.com/video/BV1XdkJYsEqz?p=19)<br><br>百度AI生成，内容仅供参考