在 Vue 3 的 Proxy 拦截器中，使用 `Reflect.get(target, key, receiver)` 而不是 `target[key]` 或 `Reflect.get(target, key)`，核心目的是修正 `this` 指向，确保在涉及继承或Getter 依赖上下文时逻辑正确。


官方文档:
- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect]
- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get]

---
  
### 一、核心优势：修正 `this` 指向

1. 场景重现

假设原始对象 `target` 包含一个 getter，该 getter 内部引用了 `this`：

```javascript
const target = {
  _name: 'Vue',
  get name() {
    // 这里的 this 指向谁至关重要
    return this._name; 
  }
};
  
const proxy = new Proxy(target, {
  get(t, key, receiver) {
    // ❌ 错误写法：target[key] 或 Reflect.get(target, key)
    // 当访问 proxy.name 时，getter 中的 this 会指向 target（原始对象）
    // 如果 target 被代理，而 _name 也在代理中，这可能导致递归错误或跳过代理逻辑

    // ✅ 正确写法：Reflect.get(target, key, receiver)
    // receiver 是 proxy 实例。getter 中的 this 将指向 proxy
    return Reflect.get(target, key, receiver);
  }
});

```

2. 为什么 `this` 指向 `receiver` (Proxy) 很重要？

- ==**保持响应式链路==**：如果 `target._name` 也是一个响应式属性（例如在嵌套对象中），当 `getter` 中的 `this` 指向 `proxy` 时，访问 `this._name` 会再次触发 Proxy 的 `get` 陷阱，从而正确收集依赖。

- **==避免“脱节”==**：如果 `this` 指向原始对象 `target`，则 `this._name` 的访问不会经过 Proxy，导致依赖丢失或无法触发更新。

### 二、对比分析

| 写法 | 代码示例 | this 指向 | 后果 |
| --- | --- | --- | --- |
| ‌推荐‌ | Reflect.get(target, key, receiver) | ‌receiver (Proxy)‌ | ✅ Getter 中的 this 指向代理对象，确保内部访问依然经过 Proxy 拦截，维持响应式完整性。 |
| ‌错误‌ | target[key] | ‌target (Raw)‌ | ❌ Getter 中的 this 指向原始对象。若原始对象内部有其他响应式属性，将‌绕过 Proxy‌，导致依赖收集失败。 |
| ‌不完整‌ | Reflect.get(target, key) | ‌target (Raw)‌ | ❌ 同上，缺少第三个参数 receiver，默认行为等同于 target[key]。 |
  

### 三、Vue 3 源码中的实际意义

在 Vue 3 的 `baseHandlers.ts` 中：

```typescript
function createGetter(isReadonly = false, shallow = false) {
  return function get(target: Target, key: string | symbol, receiver: object) {
    // ...
    // 使用 receiver 确保 getter 内部的 this 指向代理对象
    const res = Reflect.get(target, key, receiver);

    // 如果 res 是对象，继续将其转换为响应式（懒代理）
    if (isObject(res)) {
      return isReadonly ? readonly(res) : reactive(res);
    }
    return res;
  }
}
```

- ==继承场景==：如果 `proxy` 被另一个对象继承（`const child = Object.create(proxy)`），访问 `child.name` 时，`receiver` 会是 `child`。使用 `Reflect.get` 能确保 getter 中的 `this` 指向 `child`，符合 JavaScript 原型链的预期行为。

- ==一致性==：`Reflect.get` 是 `Proxy` 陷阱的标准搭档，它提供了与默认行为完全一致但可定制的执行方式。


### 疑问

==问：==
既然this指向receiiver也就是proxy，那return的访问 Reflect.get(target, key, receiver)时候是不会又回触发proxy的get(); 那这样依赖岂不是会形成递归调用了，无限循环。

==答：==
完全不会出现无限递归的情况，这个机制的执行逻辑是经过 Proxy 规范严格设计的，我们可以逐层拆解背后的运行==原理==：
#### 1. 核心边界：Reflect.get 不会二次触发当前 Proxy 的 get 陷阱

当你在 Proxy 的 `get` 拦截器内部执行 `Reflect.get(target, key, receiver)` 时，这个操作不会再次触发同一个 proxy 的 get 陷阱。
因为这里的第一个参数 `target` 是「原始的未被代理的普通对象」，不是 Proxy 实例本身。`Reflect.get` 是直接操作这个原始对象的底层属性，相当于绕过了外层 Proxy 的拦截逻辑，直接读取原始对象上的属性值。

#### 2. 完整执行链路拆解

以你示例中的代码，当访问 `proxy.name` 时，整个运行流程是绝对线性、没有任何循环的：
1.  你访问 `proxy.name`，触发外层 Proxy 的 `get` 陷阱，传入三个参数：原始对象 `t`（也就是最开始定义的普通对象 `target`）、属性名 `'name'`、接收器 `receiver`（也就是这个 proxy 实例本身）。
2.  在拦截器内部执行 `Reflect.get(target, key, receiver)`，直接对原始普通对象 `target` 读取 `name` 属性，不会再走外层 Proxy 的拦截。
3.  原始对象的 `name` 是一个 getter，由于我们传入了第三个参数 `receiver=proxy`，所以这个 getter 内部的 `this` 被绑定为 proxy 实例，而不是原始对象 target。
4.  getter 中执行 `return this._name`，此时才是第一次通过 proxy 实例访问 `_name` 属性，这会再次触发同一个 proxy 的 get 陷阱，但这是一次全新的、访问不同属性（`_name`）的合法拦截，并不是重复执行访问 `name` 的逻辑。
5.  第二次触发的 get 陷阱中，会执行 `Reflect.get(target, '_name', receiver)` 直接从原始对象中取出 `'Vue'` 字符串，返回后整个流程结束。
  
#### 3. 为什么不会死循环？

无限递归的必要条件是「操作在相同参数下不断重复调用自身」，但这个场景里不存在这个前提：
- 第一次触发 get：访问的属性是 `name`，执行后走到 getter 内部。
- 第二次触发 get：访问的属性是完全不同的 `_name`，读取到一个普通字符串，没有后续的 getter 逻辑，直接终止返回。
整个链路是两个不同属性的串行访问，永远不会回到“读取 `name`”的起始调用点，自然不可能形成死循环。
  
#### 4. 唯一可能触发循环的特殊情况

如果 getter 内部写了显式递归访问自身的逻辑，才会出现无限循环：

```javascript
const target = {
  get name() {
    return this.name // 这里强制反复读取 name 属性，才会无限触发 get 陷阱
  }
}
```

但这是代码自身的逻辑错误，和 `Reflect.get` 的写法无关，哪怕你不用 Proxy，直接执行 `target.name` 也会直接在原始对象上触发无限递归。


<br><br>百度AI生成，内容仅供参考
### 总结


使用 `Reflect.get(target, key, receiver)` 的唯一且核心好处是：确保在访问属性时，如果该属性是一个 Getter，Getter 内部的 `this` 能够正确指向代理对象（Receiver），而不是原始对象（Target）。 这对于维持 Vue 3 响应式系统的完整性和正确处理继承关系至关重要。<br>参考资料<br>[1] [Reflect.get中第三个参数receiver的作用是什么以及怎么用它修正继承关系下的this指向 - php中文网](https://www.php.cn/faq/2857717.html)<br>[2] [Reflect中怎么动态调用基类的被重写方法:利用Reflect.get和正确的receiver实现super调用 - php中文网](https://www.php.cn/faq/2886338.html)<br>[3] [如何用 Reflect.get 配合第三个参数实现对包含 Getter 的对象属性的安全访问 - php中文网](https://www.php.cn/faq/2350037.html)<br>[4] [如何利用 Reflect.get 配合 Receiver 参数实现对 Proxy 代理对象原型属性的正确访问 - php中文网](https://www.php.cn/faq/2454525.html)<br>[5] [Reflect之receiver参数 - cary](http://zhuanlan.zhihu.com/p/668451280)<br>[6] [javascript Reflect对象_怎样简化反射操作 - PHP中文网](https://m.php.cn/faq/1953882.html)<br>[7] [JavaScript Reflect是什么_它提供了哪些功能【教程】 - php中文网](https://www.php.cn/faq/2009352.html)<br>[8] [Reflect:打开 JavaScript 引擎的接线盒 - 简书社区](https://www.jianshu.com/p/e330366938ac)<br>[9] [JavaScript中的反射魔法:揭秘Reflect对象的核心方法(上) - CSDN博客](https://blog.csdn.net/JakeMa1024/article/details/148825836)<br>[10] [大厂面试题分享(纯干货) - 前后端面试指南](http://zhuanlan.zhihu.com/p/1908914942951786362)<br>[11] [Reflect.get 的性能表现与优化 - PHP中文网](https://m.php.cn/faq/2793719.html)<br>[12] [[javascript核心-01]彻底梳理清楚Proxy 代理与Reflect反射 - 腾讯云](https://cloud.tencent.com/developer/article/2296076)<br>[13] [ES6之Reflect详解 - 腾讯云](https://cloud.tencent.com/developer/article/2359563)<br>[14] [ES6 教程 - 腾讯云](https://cloud.tencent.com/edu/learning/course-1992-23512)<br>[15] [如何利用 Reflect.get/set 的 receiver 参数 解决代理对象中的 this 指向问题 - php中文网](https://www.php.cn/faq/2456688.html)<br>[16] [如何利用 Reflect.get 实现对象属性读取 - php中文网](https://www.php.cn/faq/2841514.html)<br>[17] [Reflect.get 如何正确处理对象的 getter - php中文网](https://www.php.cn/faq/2846583.html)<br>[18] [每日一题: 细说es6中的Reflect - 博客园](https://dev-preview.cnblogs.com/never404/p/17702974.html)<br>[19] [Reflect.get() 详细介绍,并给出例子说明 - 51CTO博客](https://blog.51cto.com/u_14724733/10581514)<br>[20] [什么是Reflect?Reflect的静态方法 - php中文网](https://www.php.cn/faq/1471332.html)<br>[21] [在es6 Proxy中,推荐使用Reflect.get而不是target[key]的原因 - CSDN博客](https://blog.csdn.net/qq_34629352/article/details/114210386)<br>[22] [let obj = { foo: 1 }; 为什么Reflect.get(obj, ‘foo‘, { foo: 2 }); // 输出 1? - CSDN博客](https://blog.csdn.net/2501_92798394/article/details/150066091)<br>[23] [如何利用 Reflect.get 配合 Proxy 实现对继承属性访问的细粒度权限校验 - php中文网](https://www.php.cn/faq/2424176.html)<br>[24] [手把手实现vue响应式原理 - Reflect 中receiver的作用(五) - CSDN博客](https://blog.csdn.net/frontendchen/article/details/121940628)<br>[25] [Vue3源码 结构 讲解 - liliang](http://zhuanlan.zhihu.com/p/2021338781081501721)<br>[26] [5.1.1 使用方式 - 清华大学](http://www.tup.tsinghua.edu.cn/upload/books/yz/097169-01.pdf)<br>[27] [【Java基础】JavaCore核心-反射技术 - 腾讯云](https://cloud.tencent.com/developer/article/2292273)<br>[28] [JavaScript 中对象 API 怎么使用 Reflect.ownKeys 获取所有自有键 - php中文网](https://www.php.cn/faq/2913658.html)<br>[29] [JavaScript 中 Reflect 怎么替代传统的 Object 方法调用 - php中文网](https://www.php.cn/faq/2901426.html)<br><br>百度AI生成，内容仅供参考