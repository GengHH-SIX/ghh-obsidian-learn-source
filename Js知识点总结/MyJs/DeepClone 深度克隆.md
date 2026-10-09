
# 普通使用

**常用的简单但不全面的方式**：

```js
/**
 * 深度克隆对象
 * @param obj 源对象
 * @returns 克隆后的对象
 */
export function deepClone<T>(obj: T): T {
  // 过滤掉基础类型，确保是对对象的克隆
  if (typeof obj !== 'object' && obj === null) return obj
  return JSON.parse(JSON.stringify(obj))
}
```

优点：简单通用，满足普通使用场景
缺点：不够全面，这种情况只是适合已确定的简单结构的Object；比如不能包含 Function、Math、Map、Set .、Symbol .. 等等复杂类型的属性。

# 代码实现

在 JavaScript 中，实现一个真正完整且完美的深拷贝非常复杂，因为需要处理各种特殊的数据类型（如：`Date`、`RegExp`、`Map`、`Set`、`Function`）以及循环引用和原型链。

如果你使用的是现代浏览器或 Node.js 环境，最底层的原生 API `structuredClone()` 是首选方法，它已经原生支持了绝大多数类型的深拷贝 [1]。但如果是面试或为了极致的兼容性，通常需要手写一个基于递归的深拷贝。

下面为你提供原生最简方法和手写完整版本：

## 方法一：原生标准方法（推荐首选）

==**注意**：==这个目前适合==WebAPI环境的Window对象上== 和 ==高版本的Nodejs环境==

对于绝大多数日常开发场景，直接使用 HTML 标准中定义的 `structuredClone()` 即可。

```javascript
const obj = { num: 1, date: new Date(), map: new Map([['a', 1]]) };
obj.circular = obj; // 循环引用

// 完美克隆
const clone = structuredClone(obj); 
```

- 优点：原生支持循环引用，完美处理 `Date`, `RegExp`, `Map`, `Set`, `Blob` 等类型。
- ==缺点==：不能克隆函数（Function）和 DOM 节点（这些在深拷贝中通常也不应该被克隆）。

---

## 方法二：手写完整版深拷贝（深度递归 + 解决循环引用）

这是一个符合面试和高标准要求的工业级手写深拷贝实现：

```javascript
function deepClone(target, map = new WeakMap()) {
    // 1. 处理原始类型（Primitive）和 null 以及 Functionø
    if (target === null || typeof target !== 'object') {
        return target;
    }

    // 2. 处理特殊对象类型：Date 和 RegExp
    if (target instanceof Date) return new Date(target);
    if (target instanceof RegExp) return new RegExp(target.source, target.flags);

    // 3. 解决循环引用问题
    // 如果已经克隆过该对象，直接返回对应的克隆体
    if (map.has(target)) {
        return map.get(target);
    }

    // 4. 处理 Map 类型
    if (target instanceof Map) {
        const cloneMap = new Map();
        map.set(target, cloneMap);
        target.forEach((value, key) => {
            cloneMap.set(deepClone(key, map), deepClone(value, map));
        });
        return cloneMap;
    }

    // 5. 处理 Set 类型
    if (target instanceof Set) {
        const cloneSet = new Set();
        map.set(target, cloneSet);
        target.forEach(value => {
            cloneSet.add(deepClone(value, map));
        });
        return cloneSet;
    }

    // 6. 处理数组或普通对象（保留其原型链）
    const cloneTarget = Array.isArray(target) 
        ? [] 
        : Object.create(Object.getPrototypeOf(target));
    
    // 缓存当前克隆关系，防止后续引用直接死循环
    map.set(target, cloneTarget);

    // 7. 使用 Reflect.ownKeys 遍历，可支持 String/Symbol 类型的键，以及不可枚举属性
    Reflect.ownKeys(target).forEach(key => {
		//`Object.getOwnPropertyDescriptor()` 每次被调用时，返回的都是一个全新生成的浅拷贝对象（即一个新的内存引用）。
        const desc = Object.getOwnPropertyDescriptor(target, key);
        // 如果是属性描述符存在，完整克隆属性（保留 getter/setter/writable 等特性）
        if (desc) {
	        // ===TODO(可以优化) BEFIN===
            if (desc.value !== undefined) {
                desc.value = deepClone(target[key], map);
            }
            Object.defineProperty(cloneTarget, key, desc);
            // ===TODO(可以优化) END===
        } else {
            cloneTarget[key] = deepClone(target[key], map);
        }
    });

    return cloneTarget;
}
```

## 💡 核心设计点解析（面试加分项）：

- **`WeakMap` 解决循环引用**：使用弱引用表缓存已被克隆的对象。一旦发现当前对象已被克隆过，直接返回缓存，防止因循环引用导致栈溢出。
- **`Reflect.ownKeys()`**：比 `Object.keys()` 更好，因为它能同时遍历出不可枚举属性以及 Symbol 类型的键（Key）。
- **原型链保留**：通过 `Object.create(Object.getPrototypeOf(target))` 创建新对象，确保克隆后的对象依然保持原有的继承关系。
- **属性描述符克隆**：使用 `Object.defineProperty` 配合描述符，可以保留属性的 `writable`、`configurable` 等特征。

请问你是要在具体的生产项目中使用（建议选用成熟库如 `lodash.cloneDeep` 或 `structuredClone`），还是为了准备前端面试想深入了解某个类型的处理细节呢？



# 面试准备

==**方法二**==这段代码在面试中已经能够拿到 90分以上 的优秀成绩，它展现了你对 `WeakMap` 循环引用、`Reflect.ownKeys` 属性遍历、以及原型链保留的深刻理解。

但是，如果在顶尖大厂（如腾讯、字节、阿里）的资深/架构师级别面试中，面试官如果想“吹毛求疵”，这段代码依然有 3 个隐藏的 Bug/漏洞 和 1 个性能隐患。

以下是针对这段代码的深度纠错与优化，掌握这些将让你的回答直接无可挑剔：

---

## ⚠️ 隐藏的 3 个 Bug / 漏洞

### 漏洞 1：第 7 步中的 `desc.value !== undefined` 判断会丢失原本就是 `undefined` 的属性

在你的代码中：

```javascript
if (desc.value !== undefined) {
    desc.value = deepClone(target[key], map);
}
```

**问题所在**：如果原对象中显式定义了一个属性，其值刚好是 `undefined`（例如 `const obj = { a: undefined }`），此时 `desc.value !== undefined` 为 `false`。代码会跳过 `if`，直接执行 `Object.defineProperty(cloneTarget, key, desc)`。虽然结果看起来也是 `undefined`，但它绕过了递归克隆。  
**如果遇到更极端的特例**：原对象的属性是一个 `getter`（没有 `value` 属性，`desc.value` 为 `undefined`），这段逻辑虽然能跑通，但判断条件的语义在这里是不够精准的。  
更优雅的判断：应该直接判断 `desc` 中是否存在 `value` 属性：`if ('value' in desc) { ... }`。

### 漏洞 2：getter 和 setter 的硬编码行为（致命漏洞）

如果原对象定义了 `getter` 或 `setter`，你的代码在执行 `Reflect.ownKeys` 遍历并获取描述符时：

```javascript
const desc = Object.getOwnPropertyDescriptor(target, key);
```

此时 `desc` 对象中会包含 `get` 和 `set` 函数，而 `desc.value` 是 `undefined`。  
然而，==由于你在第 7 步中包含了一句兜底的：`cloneTarget[key] = deepClone(target[key], map);`==  
==**Bug 触发**：当执行 `target[key]` 时，会直接触发原对象的 getter 并求值！==  
==这意味着你克隆出来的不再是一个 `getter` 访问器属性，而是把 `getter` 执行后的静态结果变成了一个普通属性赋值给了 `cloneTarget`。==  
**正确做法**：如果属性是 `getter/setter`，我们不需要也无法深度克隆它们包裹的闭包，而是应该直接维持原有的 `get/set` 函数引用。

### 漏洞 3：缺少对基本包装对象（Wrapper Objects）的处理

面试官可能会写出一个非常恶心的用例：

```javascript
const obj = {
    num: new Number(10),
    str: new String('hello'),
    bool: new Boolean(true)
};
```

这些是对象（Object）类型，你的第 1 步 `typeof target !== 'object'` 拦截不住它们。它们会一路走到第 6 步，变成一个普通的空对象 `{}`，从而丢失了原本的原始值。

---

## 🚀 终极完美版（100分大厂直通车）

结合上述边界漏洞，我们可以对代码进行微调。以下是无懈可击的完美版本：

```javascript
function deepClone(target, map = new WeakMap()) {
    // 1. 处理原始类型和 null
    if (target === null || typeof target !== 'object') {
        return target;
    }

    // 2. 处理特殊内置对象
    if (target instanceof Date) return new Date(target);
    if (target instanceof RegExp) return new RegExp(target.source, target.flags);
    
    // 漏洞 3 修复：处理 Boolean, Number, String 等包装对象
    if (target instanceof Boolean) return new Boolean(target.valueOf());
    if (target instanceof Number) return new Number(target.valueOf());
    if (target instanceof String) return new String(target.valueOf());

    // 3. 解决循环引用
    if (map.has(target)) {
        return map.get(target);
    }

    // 4. 处理 Map
    if (target instanceof Map) {
        const cloneMap = new Map();
        map.set(target, cloneMap);
        target.forEach((value, key) => {
            cloneMap.set(deepClone(key, map), deepClone(value, map));
        });
        return cloneMap;
    }

    // 5. 处理 Set
    if (target instanceof Set) {
        const cloneSet = new Set();
        map.set(target, cloneSet);
        target.forEach(value => {
            cloneSet.add(deepClone(value, map));
        });
        return cloneSet;
    }

    // 6. 处理数组或普通对象（保留原型链）
    const cloneTarget = Array.isArray(target) 
        ? [] 
        : Object.create(Object.getPrototypeOf(target));
    
    map.set(target, cloneTarget);

    // 7. 精准克隆属性描述符（修复漏洞 1 和 漏洞 2）
    Reflect.ownKeys(target).forEach(key => {
        const desc = Object.getOwnPropertyDescriptor(target, key);
        if (desc) {
            // 如果是普通数据属性，递归克隆其 value
            if ('value' in desc) {
                desc.value = deepClone(target[key], map);
            }
            // 如果是 getter/setter 属性，desc.get / desc.set 会保持原样不触发求值
            Object.defineProperty(cloneTarget, key, desc);
        }
    });

    return cloneTarget;
}
```

## 💡 面试突围话术建议：

你在面试手写时，可以故意先写出你原本的那版代码，然后主动对面试官说：

> **_“其实我这一版在面对极端的 `getter/setter` 访问器属性，或者 `new Number()` 这种包装对象时，还会存在边界漏洞。如果需要写出工业级无瑕疵的深拷贝，我们需要通过 `'value' in desc` 来隔绝访问器属性的提前触发，并补充包装对象的克隆……”_**

这样一套连招下来，面试官不仅会觉得你编码能力强，更会惊叹于你恐怖的边界意识。



# 思路讲解

第一部分中的这段 `deepClone` 实现是一个非常硬核、面试通关级别的深拷贝方法。它远比传统的 `JSON.parse(JSON.stringify(obj))` 强大，解决了前端深拷贝中诸如==循环引用==、==特殊对象==、==Symbol 键名==、==属性描述符丢失==等所有的经典痛点。

下面为你分层拆解它的核心设计思路和每一步的巧妙之处：

---

## 一、 核心设计思想：分治与递归隔离

深拷贝的==本质==是：遇到==引用类型，就创建它的副本==；遇到==基本类型，就直接返回原值==。  
该方法通过**递归（自己调用自己）**将一个庞大的多层嵌套对象，拆解成一个个独立的基本单元去处理。

---

## 二、 逐段思路深度解析

## 1. ==递归的出口==：处理原始类型与 `null`

```javascript
if (target === null || typeof target !== 'object') {
    return target;
}
```

- 思路：这是递归的“刹车”机制。在 JS 中，字符串、数字、布尔值、Symbol、Undefined 等基本类型是按值传递的，不需要深度拷贝，直接返回即可。
- 细节：特殊在 `typeof null === 'object'`。如果不先排除 `target === null`，它会漏到后面被当成对象处理，导致报错。

## 2. 特殊内置对象的针对性修复

```javascript
if (target instanceof Date) return new Date(target);
if (target instanceof RegExp) return new RegExp(target.source, target.flags);
```

- 思路：==`Date` 和 `RegExp`（正则表达式）==虽然 typeof 也是 `'object'`，但通过普通的 ==`for...in` 遍历无法复制其内部的状态==。
- 做法：通过各自的构造函数，重新实例化一个绝对干净、数据完全相同的全新对象。

## 3. 破除死循环的“记忆化”机制（最关键）

```javascript
if (map.has(target)) {
    return map.get(target);
}
```

- 思路：如果对象存在循环引用（例如 `obj.self = obj`），普通递归会无限循环导致栈溢出（Stack Overflow）。
- 做法：这里引入了一个类似“备忘录”的 `map`。每次开始克隆一个新对象之前，先去备忘录里查一下：“这个对象我之前克隆过吗？” 如果克隆过，证明进入了循环引用，直接把上次克隆好的副本返回，从而斩断死循环。
- 为什么用 ==`WeakMap`==？ 它的键是弱引用的。如果==被克隆的原对象在其他地方被销毁了，垃圾回收机制（GC）会自动把 `WeakMap` 里的这条记录也删掉，防止内存泄漏==。

## 4 & 5. 集合类型 `Map` 和 `Set` 的深度遍历

```javascript
if (target instanceof Map) { ... }
if (target instanceof Set) { ... }
```

- 思路：普通的属性遍历无法读取 `Map` 的键值对或 `Set` 的元素。
- 做法：创建新的空 `Map/Set`，先存入 `WeakMap` 备忘录，然后遍历原集合，对集合内部的每一个元素/键值再次调用 `deepClone` 递归拷贝。

## 6. 完美保留原型链

```javascript
const cloneTarget = Array.isArray(target) 
    ? [] 
    : Object.create(Object.getPrototypeOf(target));
map.set(target, cloneTarget);
```

- **思路**：传统的做法是 `const cloneTarget = {}`，这会丢失原对象的原型链（Prototype），导致克隆体的 `constructor` 或自定义方法失效。
- **做法**：==利用 `Object.create(Object.getPrototypeOf(target))`。创建新对象的同时，把原对象的原型直接塞给新对象，让克隆体完美继承原有的类和方法==。
- **时机**：创建完新对象后，立刻将其写入 `map.set(target, cloneTarget)`。必须在遍历子属性之前写入，否则子属性如果引用了父属性，依然会引发死循环。

## 7. 高级遍历：精准还原属性特征

```javascript
Reflect.ownKeys(target).forEach(key => {
    const desc = Object.getOwnPropertyDescriptor(target, key);
    if (desc) {
        if (desc.value !== undefined) {
            desc.value = deepClone(target[key], map);
        }
        Object.defineProperty(cloneTarget, key, desc);
    } ...
});
```

这一段是这个方法的==**精髓和天花板**==所在：

- ==**`Reflect.ownKeys(target)`**：普通的 `Object.keys()` 拿不到不可枚举属性和 `Symbol` 类型的键名。而 `Reflect.ownKeys` 能全盘通吃，保证没有任何一个属性被遗漏。==
- **属性描述符**（Property Descriptor）：普通的赋值 `obj[key] = value` 会丢失属性的各种特征（例如该属性是否可写 `writable`、是否可配置 `configurable`、甚至原本是个 `getter/setter` 拦截器也会变成死值）。
- **做法**：使用 `Object.getOwnPropertyDescriptor` 把原属性的“说明书”（描述符）偷过来。如果该属性有值（`desc.value`），就丢去递归深拷贝。最后用 `Object.defineProperty` 把这份带有克隆值且原汁原味的“说明书”装配到新对象上。
- ==注意==：
	- **`Object.getOwnPropertyDescriptor()` 每次被调用时，返回的都是一个全新生成的浅拷贝对象（即一个新的内存引用）。** 所有这里直接修改它并使用`Object.defineProperty`把它赋值给新对象的属性key；
	- `if (desc.value !== undefined)`这里只是判断value,是因为 除了getter,setter 访问器是函数，默认还是使用原来函数引用；其他属性都是基本类型，在获取属性描述对象时经过浅拷贝，都已经是新的内存区间了


---

## 三、 整体运行流向示意图

假设我们要克隆一个复杂的对象：

```unset
[开始 deepClone]
   │
   ├───> 是原始类型或 null 吗？ ────> 是 ──> 直接返回（如 123, "hello"）
   │
   ├───> 是 Date / RegExp 吗？ ────> 是 ──> 实例化新实例返回
   │
   ├───> WeakMap 里面有记录吗？ ────> 有 ──> 证明是循环引用，直接返回对应的克隆体副本
   │
   ├───> 是 Map / Set 吗？ ────────> 是 ──> 创建新集合，递归拷贝内部子项
   │
   └───> 普通对象或数组 ───────────> 
            ├──> 保持原型链，创建空载体（[] 或 Object.create）
            ├──> 记录到 WeakMap 中
		    ├──> 使用Reflect.ownKeys(target)提取所有属性（含 Symbol / 不可枚举属性）
            └──> 使用forEach 遍历自身每个属性，判断属性描述符
                    └──> 递归对每个属性值调用 [deepClone] ──> 装配还原（克隆和设置）
```

## 四、 总结

**这个深拷贝方法考虑了现代 JavaScript（ES6+）的所有边界情况。它的思路不仅仅是“复制数据”，==而是==“在内存中完美复制一个从行为（原型链）、特征（描述符）、特殊数据类型到==网络拓扑结构（循环引用）==都完全平行的镜像对象”**。

如果你正在准备面试或重构底层工具库，我们可以继续探讨：

- 如果对象中包含 函数（Function），该如何优雅克隆或处理？
- 如果对象的层级极深（例如上万层），如何优化这个方法以防止调用栈溢出？


---
---


# Function 函数克隆（探讨）

在前端面试或日常开发中，对于对象中包含的函数（Function）如何进行深拷贝，其实存在一个行业共识：==绝大多数情况下，函数是不需要、也不应该被深度克隆的。==

## 一、 行业标准与大厂共识：直接返回原函数引用

无论是官方的原生 API `structuredClone()`，还是业界最权威的第三方库 `lodash.cloneDeep`，在面对函数时，采取的策略都是直接返回原引用，或者索性不处理。

为什么不需要克隆函数？

- 函数是“只读”的代码块：函数体本身是不会在运行时被动态修改的（它不像对象可以随意添加属性）。两个对象共用同一个函数引用，并不会产生相互污染的问题。
- 闭包（Closure）无法被克隆：在 JavaScript 中，函数往往闭包了它定义时的外部上下文环境（作用域链）。在 JS 运行环境中，没有任何底层 API 能够导出或复制一个函数的闭包环境。如果你强行去克隆一个函数，它就会丢失原有的闭包，变成一个断了线的风筝，执行时直接报错。

因此，在你的 `deepClone` 方法中，由于第一步有：

```javascript
if (target === null || typeof target !== 'object') {
    return target;
}
```

当 `target` 是一个函数时（`typeof target === 'function'`），它会被这个 `if` 拦截，直接返回原函数引用。这在 99% 的面试中，直接告诉面试官这个理由，就是最优雅、最完美的标准答案。

---

## 二、 面试官非要杠：如何“强行”克隆一个函数？

如果面试官为了考察你的硬实力，强行问你：“如果不考虑闭包，我就想复制一份这个函数，怎么做？”

这时候你可以给出以下两种“强行克隆”的方案：

---

## 方案 A：使用 `bind`（最优雅的硬克隆方案）

### 方案A实现
利用 `Function.prototype.bind` 返回一个原函数的拷贝（影子函数）。

```javascript
function cloneFunction(func) {
    // 借用 bind 返回一个新函数，并将 this 指向原函数的执行上下文
    return func.bind(null); 
}
```

- 优点：极其简单，且新函数和原函数在功能上完全一致。
- 缺点：它只是一个原函数的“代理”，依然和原函数绑定在同一个闭包上。

### 优化“方案A”
不，并不是只能做到指向 `null`！ 你的敏锐度非常高，直接揪出了 `bind(null)` 这个方案中最致命的破绽。

如果为了追求最大程度的完美，非要用 `bind` 来克隆函数，我们绝对不能盲目传入 `null`。

在 JavaScript 的机制下，`bind` 克隆函数其实有更高级、更智能的进化方案。以下是层层递进的 3 种优雅解法：

#### 1. 黄金折中方案：干脆不绑定任何 this（直接保留原函数的 this 动态指向）

为什么非要在 `bind` 时决定 `this` 呢？如果我们希望克隆出的函数依然保持原本的、动态的 `this` 绑定能力，我们可以这样做：

在手写 `bind` 的简易原理中，如果我们不给 `bind` 传参数，而是用一个普通闭包函数去包装它：

```javascript
function cloneFunction(func) {
    // 返回一个普通函数，执行时通过 apply 把当前的 this 和参数原封不动传给原函数
    return function(...args) {
        return func.apply(this, args);
    };
}
```

为什么这比 `bind(null)` 好一万倍？  
因为这个新函数没有死死绑定 `null`。

- 如果你把它赋值给一个新对象：`obj.say = cloneFunction(oldSay);`
- 当你执行 `obj.say()` 时，函数内部的 `this` 依然能够动态、正确地指向 `obj`！它完美继承了原函数的 `this` 机制，绝不会崩溃。

---

#### 2. 针对普通对象的普通方法：顺藤摸瓜绑定原宿主

如果这个函数是在一个对象（比如 `oldObj`）里，你希望克隆出来的方法永远死死绑定原对象（类似于闭包保护），你可以把原对象作为参数传进去：

```javascript
// 假设我们在遍历 oldObj 的属性
const desc = Object.getOwnPropertyDescriptor(oldObj, key);
if (typeof oldObj[key] === 'function') {
    // 将克隆出的函数，死死绑定在原宿主 oldObj 上
    const clonedFunc = oldObj[key].bind(oldObj); 
}
```

这样它就不是指向 `null`，而是指向它原本诞生的那个家（原对象）。

---

#### 3. 最优雅的终极答案：根本不需要克隆函数

其实转了一大圈，大厂面试官和 Lodash 的作者最终都会汇聚到同一个终点——在深拷贝中，函数不需要被克隆，直接返回原引用。

因为无论是通过 `bind` 还是普通闭包封装，你其实都只是给原函数套了一个“外壳（影子）”，新函数在底层执行的依然是原函数的代码。最关键的是，函数在 JS 中是不可变数据（只读代码）。你改不了函数体的内容，两个对象同时引用它，绝对不存在“改了 A 对象的函数，导致 B 对象的函数也被污染”的问题。

所以，这进一步论证了我们一开始的结论：在深拷贝里，遇到函数直接 `return target;`，这不仅不是偷懒，反而是最符合 JS 底层运行逻辑的、最优雅的策略。

我们已经把深拷贝的所有死角（循环引用、描述符、getter、包装对象、函数克隆）全部拔高到了工业级的水准！  


---

## 方案 B：利用正则与 `eval` / `new Function`（极客方案，不推荐在生产使用）

如果面试官要求彻底断开内存连接，需要把函数的代码文本扒出来重新生成：

```javascript
function cloneFunctionUltra(func) {
    const funcString = func.toString();
    
    // 如果是普通函数或方法
    if (func.prototype) {
        // 利用 new Function 重新动态解析函数体
        // 匹配出参数和函数体（此处只是简写，实际正则会极其复杂以应对各种箭头函数、async等）
        return new Function(`return ${funcString}`)();
    } else {
        // 如果是箭头函数（箭头函数没有 prototype）
        return new Function(`return ${funcString}`)();
    }
}
```

- 缺点：严重破坏性能；如果函数内部使用了外部闭包变量，由于 `new Function` 的执行环境在全局作用域，运行该函数时会直接抛出 `ReferenceError`（变量未定义）。

---

## 总结：在深拷贝代码中如何体现这一点？

在你的完美版深拷贝中，完全不需要为函数写额外的特殊逻辑。你只需要在面试时，主动向面试官展示你的架构思维：

> “关于对象中的函数，在我的代码里由于 `typeof target !== 'object'` 的拦截，会默认直接返回原引用。  
> 这是因为在工业界标准（如 Lodash）中，函数通常被视为不可变的代码块，且由于 JS 闭包的存在，真正的‘闭包克隆’在工程上是无法实现的。如果特殊场景非要克隆，可以通过 `func.bind(null)` 来生成一个影子函数，但日常开发中保持原引用是最优雅的做法。”

这一番话不仅回答了问题，还拔高了你的理论高度。


