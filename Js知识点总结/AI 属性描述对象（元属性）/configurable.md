`configurable` 是 JavaScript 属性描述符中控制属性定义规则是否可修改的底层开关。它决定了该属性能否被删除，以及其描述符（如 `writable`、`enumerable`）能否被再次修改。

简单来说，`configurable: false` 意味着该属性的“元数据”被锁定，进入了不可逆的保护状态。

---
### 1. `configurable` 控制的三大行为


当 `configurable` 设置为 `false` 时，以下操作将被禁止：

1.  禁止删除属性
    使用 `delete` 操作符无法删除该属性。在非严格模式下返回 `false，在严格模式下会抛出 `TypeError`。

2.  禁止修改描述符特性
    不能使用 `Object.defineProperty()` 修改该属性的 `enumerable`、`configurable` 自身，也不能修改 `writable`（除非是从 `true` 改为 `false`，见下文特殊规则）。

3.  禁止属性类型转换
    不能在“数据属性”（含 `value/writable`）和“访问器属性”（含 `get/set`）之间相互转换。例如，不能将一个定义为 `value` 的属性重新定义为 `getter`。


###  2. `configurable` 与 `writable` 的特殊关系

这是一个容易混淆的细节：`configurable` 的权限高于 `writable`。

*   如果 `configurable: true`：
    即使 `writable: false`（只读），你仍然可以通过 `Object.defineProperty` 将 `writable` 改回 `true`，从而允许修改值。也可以随时将数据属性改为访问器属性。

*   如果 `configurable: false`：
    *   `writable` 的状态被“固化”。
    *   唯一允许的修改：如果当前 `writable` 是 `true`，你可以将其改为 `false`（进一步限制）。
    *   禁止的修改：如果当前 `writable` 是 `false`，你不能将其改回 `true`。也不能修改 `enumerable` 或 `configurable`。

> 总结：`configurable: false` 是一条单向街道。一旦关闭配置性，你就只能进一步限制属性（如把可写变成不可写），而不能放松限制。


###  3. 默认值的差异

`configurable` 的默认值取决于属性的创建方式，这直接影响属性的安全性：

| 创建方式                    | configurable 默认值 | 说明                                             |
| ----------------------- | ---------------- | ---------------------------------------------- |
| ‌对象字面量‌ { a: 1 }        | true             | 普通赋值，灵活，可删可改。                                  |
| ‌点号/方括号赋值‌ obj.a = 1    | true             | 同上，动态添加的属性默认也是可配置的。                            |
| ‌Object.defineProperty‌ | ‌false‌          | ‌关键区别‌：通过描述符显式定义的属性，默认是不可配置的，旨在提供更严格的控制。       |
| ‌Object.create (描述符)‌   | ‌false‌          | 同上，使用第二个参数定义属性时，默认不可配置。                        |
| ==‌全局变量 (var)‌==            | ==false==            | ==挂载在全局对象（如 window）上的 var 声明变量不可配置，防止意外删除核心环境属性。== |
  

###  4. 实际应用场景

1.  ==保护常量==
    结合 `writable: false` 和 `configurable: false`，可以创建真正的常量，防止被修改或删除。例如 `Math.PI` 就是不可写且不可配置的。

2.  锁定配置对象
    在库或框架中，防止用户意外删除或篡改核心配置项。

3.  实现响应式系统的基础
    Vue 2 等框架利用 `Object.defineProperty` 的 getter/setter 追踪数据变化。虽然主要依赖访问器描述符，但 `configurable` 确保了这些拦截逻辑不会被轻易移除或覆盖。

4.  模拟私有字段
    通过设置 `enumerable: false` 和 `configurable: false`，可以隐藏内部实现细节并防止外部代码删除或修改这些内部状态。


5. 代码示例
```javascript
const obj = {};

// 定义一个不可配置的属性

Object.defineProperty(obj, 'lockedProp', {
  value: 'secret',
  writable: true,      // 初始允许修改值
  configurable: false  // 锁定描述符
});


// 1. 尝试删除 -> 失败

delete obj.lockedProp; 
console.log(obj.lockedProp); // "secret" (依然存在)


// 2. 尝试修改值 -> 成功 (因为 writable 还是 true)

obj.lockedProp = 'new secret';
console.log(obj.lockedProp); // "new secret"

  
// 3. 尝试将 writable 改为 false -> 成功 (允许进一步限制)

Object.defineProperty(obj, 'lockedProp', {
  writable: false
});

  

// 4. 尝试将 writable 改回 true -> 报错! (不可逆)

try {
  Object.defineProperty(obj, 'lockedProp', {
    writable: true
  });
} catch (e) {
  console.log(e.message); // "Cannot redefine property: lockedProp"
}


// 5. 尝试修改 enumerable -> 报错!
try {
  Object.defineProperty(obj, 'lockedProp', {
    enumerable: true
  });
} catch (e) {
  console.log(e.message); // "Cannot redefine property: lockedProp"
}

```

  

理解 `configurable` 是掌握 JavaScript 对象底层权限控制的关键。它提供了一种“一旦设定，不可反悔”的机制，常用于构建健壮、安全的 API 和数据结构。

### 参考资料
<br>[1] [如何理解JavaScript属性描述符_如何控制对象属性的行为-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/1884392.html)<br>[2] [JavaScript 对象属性的属性标签：configurable 含义探讨-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2606711.html)<br>[3] [彻底搞懂 JavaScript 数据属性描述符：告别懵逼，进阶大神引言 在JavaScript中，理解对象的属性是非常重 - 掘金 - 掘金开发者社区](https://juejin.cn/post/7471168982600073256)<br>[4] [JavaScript中对象属性的配置性configurable的影响-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2219552.html)<br>[5] [js Object.defineProperty()详解-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/2001277)<br>[6] [js Object.defineProperty()详解-腾讯云开发者社区-腾讯云 - cloud.tencent.com.cn](https://cloud.tencent.com.cn/developer/article/2001277)<br>[7] [如何理解Object.defineProperty()?-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/949015)<br>[8] [详解Object.defineProperty方法-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1121916)<br>[9] [Object.defineProperty方法详解-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/2511279)<br>[10] [JS对象属性描述符_Configurable与Writable-js教程-PHP中文网  - PHP中文网](https://m.php.cn/faq/1739530.html)<br>[11] [属性描述对象 · Javascript教程 · 看云 - 看云](https://www.kancloud.cn/wangdaodao/javascript/1317954)<br>[12] [深入理解JavaScript_属性描述符-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/1767727.html)<br>[13] [JavaScript 中 configurable 的严谨使用指南与常见陷阱-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2775830.html)<br>[14] [js中的属性描述符 - 知乎](http://zhuanlan.zhihu.com/p/1914291421948678270)<br>[15] [JavaScript 属性描述对象 - GitHub中文社区 - www.github.best](https://www.github.best/archives/js-shu-xing-miao-shu-dui-xiang)<br>[16] [JavaScript 对象与属性描述符：从原理到实战背景：为什么要深入理解对象？ 在日常开发中，我们经常会遇到这样的困惑 - 掘金 - 掘金开发者社区](https://juejin.cn/post/7616222207400738867)<br>[17] [属性描述对象 - JavaScript 内置对象（标准库） - www.lanmper.cn](https://www.lanmper.cn/js/t10022)<br>[18] [你不容错过的JavaScript高级语法(二)-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1051760)<br>[19] [了解JavaScript对象的特殊属性-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/2055806)<br>[20] [【你不知道的JavaScript】对象属性描述符本系列为学习笔记，我将在此记录我从【你不知道的JavaScript】中获 - 掘金 - 掘金开发者社区](https://juejin.cn/post/6898684702186717197)<br>[21] [JS对象属性writable、enumerable、configurable_js种将对象设置writable-CSDN博客 - CSDN博客](https://blog.csdn.net/2401_84092360/article/details/137458658)<br>[22] [继往开来 · JavaScript 二十年 - cn.history.js.org](https://cn.history.js.org/part-4.html)<br>[23] [详解JavaScript之神奇的Object.defineProperty - 知乎 - 知乎](https://zhuanlan.zhihu.com/p/39041991)<br>[24] [JavaScript｜面向对象 or 基于对象？-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/2009002)<br>[25] [【JavaScript 教程】标准库—属性描述对象 - 微信公众平台](https://mp.weixin.qq.com/s?__biz=MjM5MDA2MTI1MA==&mid=2649090342&idx=2&sn=21499718f3952432a370619426c9b653&chksm=be5bce8b892c479dff810dc524403a0aeaf41b685437b85ab526fc2e20e16cb6b8358d1965f4&scene=27)<br>[26] [关于JavaScript属性描述对象不得不掌握的知识点_enumerable: false, writable: true, configurable: t-CSDN博客 - CSDN博客](https://blog.csdn.net/Octopus21/article/details/125829680)<br>[27] [JavaScript对象深入理解（defineProperty、get/set）_js defineproperty get set-CSDN博客 - CSDN博客](https://blog.csdn.net/doy_doy/article/details/80513467)<br>[28] [详解JavaScript 的属性标志和属性描述符-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1059236)<br>[29] [深入探讨JavaScript属性访问控制机制 - OSCHINA - 中文开源技术交流社区 - 开源中国](https://my.oschina.net/emacs_8693202/blog/17023944)<br>[30] [JavaScript——对象 - 知乎 - 知乎](https://zhuanlan.zhihu.com/p/510327692)<br>[31] [JavaScript 中属性描述符对 Reflect.defineProperty 的影响-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2806952.html)<br>[32] [深入解析JavaScript属性描述符和属性标志_configurable和enumerable-CSDN博客 - CSDN博客](https://blog.csdn.net/qq_41152573/article/details/135601275)<br>[33] [JS中的属性描述符-CSDN博客 - CSDN博客](https://blog.csdn.net/qq_37969387/article/details/148289779)<br>[34] [Object.defineProperties() - JavaScript | MDN - MDNWeb文档平台](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)<br><br>百度AI生成，内容仅供参考