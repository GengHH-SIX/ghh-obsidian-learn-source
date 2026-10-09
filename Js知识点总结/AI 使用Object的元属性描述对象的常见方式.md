
除了 `Object.create` 的第二个参数，JavaScript 中还有以下几种主要方式可以在定义对象时指定元属性（如 `writable`, `enumerable`, `configurable`）：


==**注意：**==
在JavaScript规范中，只要是通过==**属性描述符接口**==（包括`Object.create`、`Object.defineProperty`、`Object.defineProperties`）==新增属性==，‌**没有显式声明`enumerable`字段的情况下，该属性的`enumerable`属性会默认被设置为`false`**‌。

| 特性               | ‌对象字面量‌ { key: val } | ‌属性描述符‌ Object.defineProperty / Object.create |
| ---------------- | -------------------- | --------------------------------------------- |
| ‌writable‌       | true (默认可写)          | ‌false‌ (默认只读)                                |
| ‌enumerable‌     | true (默认可枚举)         | ‌false‌ (默认隐藏) 不可枚举                           |
| [[configurable]] | true (默认可配置/删除)      | ‌false‌ (默认锁定) **无法修改某些东西**                   |

---

### 1. `Object.defineProperty()`

这是最基础、最常用的单独定义属性描述符的方法。它允许你精确控制单个属性的行为。


```javascript
const obj = {};

Object.defineProperty(obj, 'myProp', {
  value: function() { return 'hello'; }, // 属性值是一个函数
  writable: false,    // 不可修改
  enumerable: true,   // 可枚举
  configurable: false // 不可删除或重新配置
});


console.log(obj.myProp()); // "hello"
obj.myProp = function() { return 'world'; }; // 严格模式下报错，非严格模式静默失败

```

  
### 2. `Object.defineProperties()`

当你需要一次性定义或修改多个属性及其元属性时，这个方法比多次调用 `defineProperty` 更高效且代码更整洁。它的第二个参数格式与 `Object.create` 的第二个参数完全一致。

  
```javascript

const obj = {};

Object.defineProperties(obj, {
  prop1: {
    value: 100,
    writable: true,
    enumerable: true,
    configurable: true
  },

  prop2: {
    value: function() { return this.prop1 * 2; },
    writable: false,
    enumerable: false, // 不可枚举
    configurable: false
  }
});

console.log(obj.prop1); // 100
console.log(obj.prop2()); // 200
console.log(Object.keys(obj)); // ["prop1"]，prop2 因为 enumerable: false 被隐藏

```


### 3. ES6 Class 中的 `static` 和 getter/setter (间接方式)

虽然 Class 语法糖主要关注的是原型链和方法定义，但它也提供了一些控制属性行为的机制，尽管不如描述符那样全面（例如 Class 中定义的方法默认是 `enumerable: false`，且不能直接通过 Class 语法设置 `writable: false`）。

*   Getter/Setter: 可以定义访问器属性，这本身就是一种特殊的元属性配置（使用 `get` 和 `set` 而不是 `value`）。
*   注意: Class 中无法直接通过语法设置 `writable` 或 `configurable`，如果需要这些精细控制，通常还是需要在构造函数或类外部结合 `Object.defineProperty` 使用。

```javascript

class MyClass {
  constructor() {
    // 在实例上定义具有特定元属性的属性
    Object.defineProperty(this, 'secret', {
      value: 'shhh',
      writable: false,
      enumerable: false,
      configurable: false
    });
  }

  // 这是一个访问器属性 (Accessor Property)，没有 value/writable
  get computedValue() {
    return 42;
  }

}

const inst = new MyClass();

console.log(inst.secret); // "shhh"

// inst.secret = 'new'; // 失败
```

  

### 4. `Object.freeze()` / `Object.seal()` / `Object.preventExtensions()`

这些方法不是用来“定义”单个属性的元属性，而是批量修改对象现有属性的元属性状态：

*   `Object.freeze(obj)`: 将所有自有属性设置为 `writable: false` 和 `configurable: false`，并阻止添加新属性。对象变为完全不可变。

*   `Object.seal(obj)`: 将所有自有属性设置为 `configurable: false`，但 `writable` 保持原样（如果原来是 true，则仍可写）。阻止添加新属性。

*   `Object.preventExtensions(obj)`: 仅阻止添加新属性，不影响现有属性的 `writable` 或 `configurable`。


### 总结对比

| 方式                           | 适用场景          | 能否设置所有元属性             | 未指定属性描述符的默认值规则                                                                                                                                         |
| ---------------------------- | ------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ‌对象字面量 {}‌                   | 快速创建简单对象      | ❌ 只能创建默认的可写、可枚举、可配置属性 | 所有属性默认自动满足：writable: true、enumerable: true、configurable: true，不存在任何锁定限制。                                                                               |
| ‌Object.create(proto, desc)‌ | 创建新对象并指定原型+属性 | ✅ 能                   | 通过第二个参数的属性描述符新增属性时，所有未显式声明的描述符字段默认值为：writable: false、enumerable: false、configurable: false。                                                            |
| ‌Object.defineProperty()‌    | 修改或定义单个属性     | ✅ 能                   | 定义/新增属性时，仅配置了value或访问器字段时，其余未声明的描述符字段默认值为：writable: false、enumerable: false、configurable: false。                                                       |
| ‌Object.defineProperties()‌  | 批量定义或修改多个属性   | ✅ 能                   | 批量定义属性时，所有属性的未声明描述符字段默认值和单属性定义规则完全一致：writable: false、enumerable: false、configurable: false。                                                            |
| ‌Object.freeze/seal‌         | 锁定对象状态        | ⚠️ 批量修改，不能自定义单个属性的具体值 | 该类方法是批量修改已有属性的状态：- 执行freeze后，所有已有属性强制变为writable: false、configurable: false，其余特性保持原有状态；- 执行seal后，所有已有属性强制变为configurable: false，writable保留原本的状态，不会被强制修改。 |

### 核心结论：

如果你需要在定义对象时精细控制元属性，`Object.defineProperties` 是 `Object.create` 之外最直接的替代方案，两者使用的描述符格式完全相同。而 `Object.defineProperty` 则适合对单个属性进行后续的微调。

###  参考资料

<br>[1] [一文读懂对象方法：Object.create、defineProperties、defineProperty 详解与案例分析_04JavaScript中级（已完结）-CSDN专栏 - CSDN下载](https://download.csdn.net/blog/column/12878856/145201995)<br>[2] [【放假月日更】Object.create() 详解-腾讯云开发者社区-腾讯云 - 腾讯云](https://cloud.tencent.com/developer/article/2054430)<br>[3] [Javascript Object.create 用法详解-云社区-华为云 - 云计算社区](https://bbs.huaweicloud.com/blogs/440367)<br>[4] [Javascript Object.create 用法详解_应用服务_华为云论坛 - 华为开发者联盟](https://developer.huawei.com/home/forum/hwc/thread-02112167888214536012-1-1.html)<br>[5] [JavaScript描述Object.create()方法的原理及作用，如何利用它创建一个具有指定原型的对象？-CSDN博客 - CSDN博客](https://blog.csdn.net/qq_37212162/article/details/146539615)<br>[6] [一文读懂对象方法：Object.create、defineProperties、defineProperty 详解与案例分析_object.defineproperties-CSDN博客 - CSDN博客](https://blog.csdn.net/chinayun_6401/article/details/145201995)<br>[7] [JavaScript中Object-create实现原型继承的语法-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2217622.html)<br>[8] [JavaScript 中原型怎么使用 Object.create() 方法以指定原型对象直接创建新对象-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/3032671.html)<br>[9] [图形系统开发预备课程：学习JavaScript——21.属性描述对象 - 知乎 - 知乎](https://zhuanlan.zhihu.com/p/8475120938)<br>[10] [一文讲解JS中Object对象一些操作方法（分享）-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/481173.html)<br>[11] [JavaScript Object对象_w3cschool - w3cschool](https://www.w3cschool.cn/nwfchn/2qyzlozt.html)<br>[12] [JavaScript浅析 -- 属性描述对象 - 简书 - 简书社区](https://www.jianshu.com/p/8b59cf6e3ebb)<br>[13] [Object.create() - JavaScript | MDN - Mozilla开发者网络（缓存/镜像）](https://developer.mozilla.org.cach3.com/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)<br>[14] [Object.create() - JavaScript | MDN - MDNWeb文档平台](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)<br>[15] [JavaScript中object对象的使用，对象常用方法总结介绍 create defineProperty assign values - 知乎 - 知乎](https://zhuanlan.zhihu.com/p/490643385)<br>[16] [详解Object.defineProperty方法-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1121916)<br>[17] [`Object.create()` 方法的参数-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1635625)<br>[18] [JavaScript 的原型方法 Object.create 用法-js教程-PHP中文网  - php中文网](https://www.php.cn/faq/2845270.html)<br>[19] [Object.create() - 知乎 - 知乎](https://zhuanlan.zhihu.com/p/70925944)<br>[20] [Object.create ，Object.assign个人学习 Object.create Object.create - 掘金 - 掘金开发者社区](https://juejin.cn/post/7212101192746008632)<br>[21] [Object.create()和Object.defineProperties()Object.defineProperty()的用法 - 掘金 - 掘金开发者社区](https://juejin.cn/post/6979494642702090270)<br>[22] [【面试题】 JS手写ES6的Object.create方法-阿里云开发者社区 - 阿里云开发者社区](https://developer.aliyun.com/article/1414004)<br>[23] [Object 对象的相关方法 · Javascript教程 · 看云 - 看云](https://www.kancloud.cn/wangdaodao/javascript/1317968)<br><br>百度AI生成，内容仅供参考