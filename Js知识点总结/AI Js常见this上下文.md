
结合你之前一直在研究 JavaScript 类的静态属性、Pinia 状态管理等前端基础内容，这里为你梳理除类体之外，JavaScript 中所有具有独立 `this` 上下文的场景和结构。

#### 一、独立 this 上下文的核心定义
独立 `this` 上下文指的是：该结构内部的 `this` 不会继承外层作用域的 `this`，而是拥有自己独立的绑定规则，其指向由调用方式决定。

---

#### 二、具体场景与结构

1. 普通函数（非箭头函数）
   - 这是最基础的独立 `this` 上下文场景。
   - 普通函数的 `this` 指向完全由调用方式决定：全局调用时指向全局对象（浏览器中为 `window`），作为对象方法调用时指向该对象，通过 `call/apply/bind` 可以手动绑定 `this`。
   - 示例：

     ```javascript

     function normalFunc() {
       console.log(this) // 独立上下文，不继承外层 this
     }
     ```

2. 对象的方法
   - 定义在对象字面量中的函数，作为对象的属性被调用时，拥有独立的 `this` 上下文。
   - 此时 `this` 指向调用该方法的对象本身，不会向外层作用域查找 `this`。
   - 示例：

     ```javascript

     const obj = {
       name: 'test',
       sayName() { // 独立 this 上下文
         console.log(this.name)
       }
     }
     obj.sayName() // 输出 ‘test’ this指向obj本身
     
     //eg: 赋值引用调用时候指向运行着（这里是window）
     const aa = obj.sayName 
     aa() // 控制台输出 '' (因为现在window.name = '') 
     ```

  

3. 构造函数
   - 通过 `new` 关键字调用的构造函数，拥有完全独立的 `this` 上下文。
   - 构造函数内部的 `this` 指向新创建的实例对象，不会受外层 `this` 影响。
   - 示例：

     ```javascript
     function Person() {
       this.name = '张三' // this 指向新实例，独立上下文
     }
     ```


4. 通过 `call/apply/bind` 绑定的函数
   - 即使是普通函数，通过这三个方法手动指定 `this` 指向后，会拥有独立的绑定上下文。
   - 绑定后的函数内部 `this` 被固定为指定值，不再受调用方式影响。
   - 示例：

     ```javascript

     const func = function() { console.log(this) }
     const boundFunc = func.bind({ value: 1 }) // 生成独立绑定的 this 上下文

     ```
  

5. DOM 事件处理函数
   - 浏览器中绑定的原生 DOM 事件回调函数，拥有独立的 `this` 上下文。
   - 事件触发时，回调内的 `this` 默认指向绑定该事件的 DOM 元素，而非外层作用域的 `this`。
   - 示例：

     ```javascript

     button.addEventListener('click', function() {
       console.log(this) // this 指向 button 元素，独立上下文
     })

     ```

  

6. 类的方法（非箭头函数）
   - 类中定义的普通实例方法，拥有独立的 `this` 上下文。
   - 方法内的 `this` 默认指向类的实例对象，脱离实例直接调用时 `this` 会丢失（严格模式下为 `undefined`）。
   - 示例：

     ```javascript

     class Demo {
       log() { // 独立 this 上下文
         console.log(this)
       }
     }
     ```

  

7. 生成器函数（`function*`）
   - 生成器函数拥有独立的 `this` 上下文，其 `this` 指向规则和普通函数完全一致。
   - 生成器返回的迭代器对象调用 `next()` 时，不会改变生成器函数内部的 `this` 指向。
   - 示例：

     ```javascript

     function* gen() {
       console.log(this) // 独立上下文，指向由调用方式决定
       yield 1
     }

     ```


---


三、反例：不具备独立 this 上下文的结构

这些结构的 `this` 会直接继承外层作用域的 `this`，不属于独立上下文场景：

- 箭头函数：完全没有自己的 `this`，直接捕获外层作用域的 `this`[[AI 箭头函数中的 this]]。

- 顶层作用域：全局环境下的 `this` 是全局对象，不存在独立上下文。

- 块级作用域：`{ }` 包裹的普通代码块，不会创建新的 `this` 上下文。<br>参考资料<br>[1] [非常实用的JavaScript Class this上下文绑定 - 远程前端brandon](http://www.bilibili.com/video/BV13B4y1B7P5)<br>[2] [JS进阶:上下文 - 鱼C-小甲鱼](http://www.bilibili.com/video/BV17b41137yx?p=10?p=38)<br>[3] [请解释一下JavaScript中的this关键字在不同上下文中的行为。 - 百度教育](https://easylearn.baidu.com/edu-page/tiangong/questiondetail?id=1832913715640462141&fr=search)<br>[4] [深入解读 JavaScript 中 `this` 的指向机制:覆盖所有场景与底层原理 - CSDN博客](https://blog.csdn.net/qq_50708187/article/details/146185345)<br>[5] [JS 中 this上下文对象的使用方式 - CSDN博客](https://blog.csdn.net/sinat_17775997/article/details/55798590/)<br>[6] [JS 中 this上下文对象的使用方式 - 博客园](https://www.cnblogs.com/imwtr/p/4766543.html)<br>[7] [JavaScript中的this用法解析及实例演示 - CSDN下载](https://download.csdn.net/blog/column/12455034/133510377)<br>[8] [JavaScript 中的 "This" - 阿张](http://zhuanlan.zhihu.com/p/15755654453)<br>[9] [JavaScript中的this, 究竟指向什么? - jzplp](https://zhuanlan.zhihu.com/p/12606133403)<br>[10] [轻松掌握:前端大师讲解Javascript箭头函数中的this奥秘 - 前端亮亮](http://www.bilibili.com/video/BV1znqHYbEfZ)<br>[11] [38-JS进阶:上下文 - 无单5](http://www.bilibili.com/video/BV1BwrwYyEe4)<br>[12] [搞懂JavaScript 的 this 绑定:从踩坑到举一反三 - 腾讯云](https://cloud.tencent.com/developer/article/2565731)<br>[13] [JavaScript 之 this 详解 - www.cloud.tencent.com](https://www.cloud.tencent.com/developer/article/1075070)<br>[14] [JS 中 this上下文对象的使用方式-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/1326918)<br>[15] [JavaScript中全局上下文、函数上下文与Eval上下文中的this怎么区分 - php中文网](https://www.php.cn/faq/2886012.html)<br>[16] [Azure Functions Node.js 开发者参考 - Microsoft](https://docs.microsoft.com/zh-cn/azure/azure-functions/functions-reference-node)<br>[17] [JavaScript 中“this”可以指向的 8 个不同的地方 - web前端开发](https://mp.weixin.qq.com/s?__biz=MjM5MDA2MTI1MA==&mid=2649125961&idx=2&sn=fa0725d5409ad037079438eb4b4620aa&chksm=be5853e4892fdaf20c47d1cb4f90c0fe4b2a9c20004649c2f3b7aa0cc9c0ecbfb17e6045b572&scene=27)<br>[18] [this(执行上下文) - CSDN博客](https://blog.csdn.net/weixin_62061392/article/details/146467234)<br>[19] [深入理解 JavaScript `this` 关键字:上下文绑定与动态行为 - CSDN博客](https://blog.csdn.net/weixin_42107409/article/details/152273188)<br>[20] [javascript 执行上下文与上下文this - CSDN博客](https://blog.csdn.net/weixin_42707287/article/details/110929905)<br>[21] [JS 中 this 在各个场景下的指向 - CSDN博客](https://blog.csdn.net/chaoren8728/article/details/100960270)<br>[22] [javascript中的this详解及应用场景 - CSDN博客](https://blog.csdn.net/teeeeeeemo/article/details/149156899)<br>[23] [一文知根:JS中this到底指的谁? - CSDN博客](https://blog.csdn.net/qq_62914927/article/details/149816495)<br>[24] [JS中this值的问题 - CSDN博客](https://blog.csdn.net/wgm_123/article/details/142382597)<br>[25] [JS总结:(二)执行上下文、this、作用域与闭包-CSDN博客 - CSDN博客](https://blog.csdn.net/weixin_30917213/article/details/99279023)<br><br>百度AI生成，内容仅供参考