**详细图解** [[图- 箭头函数大全.excalidraw]]

## 概述

箭头函数（`=>`）是 ES6 引入的重要语法特性，其核心设计目标之一是解决传统函数中 `this` 指向动态变化导致的混乱问题。与普通函数不同，箭头函数没有自己的 `this`，其 `this` 绑定遵循**词法作用域规则**——在函数定义时就确定，且永远指向定义时所在的外层作用域的 `this`，与调用方式无关。

理解箭头函数的 `this` 绑定机制，是掌握 ES6 异步编程、回调函数优化的关键，也是区分前端开发能力层级的重要考点。

## 一、普通函数与箭头函数的 this 核心差异

普通函数的 `this` 是**动态绑定**的，指向取决于函数的调用方式；而箭头函数的 `this` 是**词法绑定**的，指向定义时的外层作用域 `this`。二者差异可通过下表清晰区分：

| 特性              | 普通函数                         | 箭头函数                   |
| --------------- | ---------------------------- | ---------------------- |
| `this` 绑定时机     | 函数调用时动态确定                    | 函数定义时静态确定（词法绑定）        |
| `this` 指向依据     | 调用者（谁调用函数）                   | 定义时的外层作用域              |
| 能否修改 `this`     | 可通过 `call`/`apply`/`bind` 修改 | 不可修改，永远绑定外层 `this`     |
| 能否作为构造函数        | 可以（`new` 调用生成实例）             | 不可以（`new` 调用报错）        |
| 是否有 `arguments` | 有（类数组对象）                     | 无（需用剩余参数 `...args` 替代） |

### 代码示例：直观对比

```JavaScript
// 普通函数：this 动态绑定
const obj1 = {
  name: "普通函数对象",
  fn: function() {
    console.log(this.name);
  }
};
const outerFn = obj1.fn;
obj1.fn(); // 输出 "普通函数对象"（调用者是 obj1）
outerFn(); // 输出 undefined（调用者是全局对象/undefined）

// 箭头函数：this 词法绑定
const obj2 = {
  name: "箭头函数对象",
  fn: () => {
    console.log(this.name);
  }
};
const outerArrow = obj2.fn;
obj2.fn(); // 输出 undefined（定义时外层作用域是全局，this 指向全局）
outerArrow(); // 输出 undefined（this 无法修改）
```

## 二、箭头函数 this 的绑定规则

### 1. 核心规则：继承外层作用域的 this

箭头函数没有自身的 `this` 绑定，它会捕获定义时所在的**最近外层作用域**的 `this`，并将其作为自己的 `this`。这里的“外层作用域”可以是全局作用域、普通函数作用域、类方法作用域等，但不能是其他箭头函数（因为其他箭头函数也没有自己的 `this`）。

```JavaScript
// 外层普通函数作用域
function outer() {
  const self = this; // 保存外层 this
  const innerArrow = () => {
    console.log(this === self); // 输出 true，箭头函数 this 继承外层
  };
  innerArrow();
}

const obj = { name: "测试" };
outer.call(obj); // 外层 this 被 call 改为 obj，箭头函数 this 也随之绑定 obj
```

### 2. this 不可修改特性

无论通过 `call`、`apply`、`bind` 方法尝试修改箭头函数的 `this`，还是改变函数的调用方式，都无法改变其 `this`指向。传入的 `this` 参数会被直接忽略。

```JavaScript
const arrowFn = () => {
  console.log(this);
};

const newObj = { name: "新对象" };
arrowFn.call(newObj); // 输出全局对象（浏览器：window，Node.js：global）
arrowFn.apply(newObj); // 同上
const boundFn = arrowFn.bind(newObj);
boundFn(); // 同上，this 仍为全局对象
```

### 3. 全局作用域中的箭头函数

在全局作用域中定义的箭头函数，其 `this` 直接指向全局对象（浏览器环境为 `window`，Node.js 环境为 `global`），且永远不会改变。

```JavaScript
// 全局作用域
const globalArrow = () => {
  console.log(this === window); // 浏览器环境输出 true
};
globalArrow();

// 严格模式下无影响
'use strict';
const strictGlobalArrow = () => {
  console.log(this === window); // 仍输出 true（全局作用域严格模式 this 仍是 window）
};
strictGlobalArrow();
```

## 三、箭头函数 this 的常见场景应用

### 1. 回调函数中的 this 穿透（解决经典痛点）

传统回调函数（如 `setTimeout`、数组方法、Promise 回调）中，普通函数的 `this` 会丢失或指向异常，需通过 `var that = this` 或 `bind(this)` 规避；箭头函数可直接继承外层 `this`，完美解决该问题。

```JavaScript
const user = {
  name: "张三",
  hobbies: ["读书", "跑步"],
  printHobbies: function() {
    // 普通函数回调：this 指向全局，需 bind
    this.hobbies.forEach(function(hobby) {
      console.log(`${this.name} 喜欢 ${hobby}`); // 输出 "undefined 喜欢 读书"（错误）
    }.bind(this)); // 手动绑定外层 this

    // 箭头函数回调：自动继承外层 this
    this.hobbies.forEach(hobby => {
      console.log(`${this.name} 喜欢 ${hobby}`); // 输出 "张三 喜欢 读书"（正确）
    });
  }
};
user.printHobbies();
```

### 2. 类方法中的箭头函数（固定实例 this）

类的普通方法中，`this` 指向实例，但当方法被单独调用时（如作为事件回调），`this` 会丢失；将类方法定义为箭头函数，可永久绑定实例 `this`。

```JavaScript
class Person {
  constructor(name) {
    this.name = name;
    // 类构造函数中定义箭头函数方法，this 绑定实例
    this.sayHi = () => {
      console.log(`你好，我是 ${this.name}`);
    };
  }

  // 普通类方法（this 动态绑定）
  sayHello() {
    console.log(`你好，我是 ${this.name}`);
  }
}

const person = new Person("李四");
const { sayHi, sayHello } = person;

sayHi(); // 输出 "你好，我是 李四"（this 仍绑定实例）
sayHello(); // 输出 "你好，我是 undefined"（this 丢失，指向全局）
```

### 3. 禁止作为构造函数使用

箭头函数不能通过 `new` 关键字调用，否则会抛出 `TypeError`。原因是构造函数需要通过 `this` 指向新创建的实例，而箭头函数没有自己的 `this`，无法完成实例绑定。

```JavaScript
const ArrowConstructor = () => {};

new ArrowConstructor(); 
// 报错：TypeError: ArrowConstructor is not a constructor
```

### 4. 事件回调中的注意事项

普通函数作为 DOM 事件回调时，`this` 指向触发事件的元素；而箭头函数作为事件回调时，`this` 继承定义时的外层作用域，无法指向事件元素。

```HTML
<button id="btn">点击测试</button>
<script>
const btn = document.getElementById("btn");

// 普通函数事件回调：this 指向 btn 元素
btn.addEventListener("click", function() {
  console.log(this === btn); // 输出 true
});

// 箭头函数事件回调：this 继承外层（全局 window）
btn.addEventListener("click", () => {
  console.log(this === window); // 输出 true
  console.log(this === btn); // 输出 false
});
</script>
```

## 四、箭头函数与普通函数的 this 绑定优先级

当多种绑定规则同时存在时，需明确优先级关系，箭头函数的词法绑定优先级最高，不受其他规则影响：

1. 箭头函数：词法绑定（优先级最高，不可修改）
    
2. 普通函数：`new` 调用（构造函数绑定）> `call`/`apply`/`bind` 绑定 > 对象方法调用 > 普通函数调用（全局/undefined）
    

```JavaScript
// 普通函数优先级示例
function fn() {
  console.log(this.name);
}
const obj1 = { name: "obj1" };
const obj2 = { name: "obj2" };

fn.call(obj1); // 输出 "obj1"（call 绑定优先级高于普通调用）
const boundFn = fn.bind(obj2);
boundFn.call(obj1); // 输出 "obj2"（bind 绑定优先级高于 call）
new boundFn(); // 输出 undefined（new 绑定优先级高于 bind）

// 箭头函数不受上述规则影响
const arrowFn = () => {
  console.log(this.name);
};
arrowFn.call(obj1); // 输出全局 name（无则 undefined），词法绑定优先级最高
```

## 五、常见误区与注意事项

### 1. 误区：箭头函数可以解决所有 this 问题

箭头函数仅适用于“需要继承外层 this”的场景，若需动态 `this`（如事件回调、类的普通方法、构造函数），应使用普通函数。

### 2. 误区：==箭头函数中的 this 指向定义时的自身作用域==

箭头函数没有自身作用域的 `this`，必须向外层作用域查找，直到找到第一个普通函数（或全局作用域）的 `this`。

```JavaScript
const a = () => {
  const b = () => {
    console.log(this); // 向外层查找，找到全局 this
  };
  b();
};
a(); // 输出全局对象
```

### 3. 注意：==对象字面量中的箭头函数 this==

对象字面量中的箭头函数，其==外层作用域是全局作用域==，而非所在对象（**对象字面量不构成独立作用域** [[AI 作用域创建场景]]）。

```JavaScript
const obj = {
  name: "对象",
  fn: () => {
    console.log(this === window); // 输出 true，this 指向全局
  }
};
obj.fn(); // 并非指向 obj，在当前词法环境中绑定为window
```

### 4. 注意：严格模式对箭头函数 this 的影响

严格模式仅影响普通函数的 `this`（普通函数独立调用时 `this` 为 `undefined`），对箭头函数的 `this` 无影响——箭头函数仍继承外层作用域的 `this`。

```JavaScript
'use strict';
function strictOuter() {
  const arrow = () => {
    console.log(this); // 外层普通函数独立调用时 this 为 undefined，箭头函数也随之绑定 undefined
  };
  arrow();
}
strictOuter(); // 输出 undefined
```

## 六、总结

1. 箭头函数的 `this` 遵循==**词法绑定规则**==，定义时确定，继承外层作用域的 `this`，与调用方式无关。
2. 箭头函数的 `this` 不可通过 `call`/`apply`/`bind` 修改，也不能作为构造函数使用。
3. 核心==优势==是==解决回调函数中 `this` 丢失问题，简化异步编程逻辑==。
4. 适用场景：短回调函数、需要固定 `this` 的场景；
5. 不适用场景：构造函数、事件回调（需指向元素）、动态 `this` 场景。

掌握箭头函数的 `this` 绑定机制，需结合实际场景对比普通函数的差异，明确其设计初衷与使用边界，才能在开发中灵活运用，避免 `this` 指向错误。

## 七 易错点
```javascript
"use strict";

const obj = {
  i: 10,
  b: () => console.log(this.i, this),
  c() {
    console.log(this.i, this);
  },
};

obj.b(); // 输出 undefined, Window { /* … */ }（或全局对象）
obj.c(); // 输出 10, Object { /* … */ }
```
问：为什么调用obj.b() 指向的是window，他不是个箭头函数吗？不应该是由词法环境绑定，也就是外层对象obj吗？？？
答：obj不构成作用域，他只是个数据的集合
- ‌**对象**‌：是数据的集合，‌**不创建作用域**‌。
- ‌**函数**‌：是代码的执行单元，‌**创建函数作用域**‌。
- ‌**块**‌：`{}` 中包含 `let/const/class` 时，‌**创建块级作用域**‌。


 JavaScript 箭头函数词法绑定的经典误区，核心原因是‌**对象字面量(obj)本身不构成独立的函数作用域**‌，箭头函数捕获的外层 `this` 并不是你以为的 `obj`。

### 一、核心原因拆解

1. ‌**箭头函数的词法绑定规则**‌  
    箭头函数的 `this` 完全由‌**定义它时所处的外层普通函数作用域**‌决定，没有自己的独立 `this`，不会被调用方式改变。
    
2. ‌**对象字面量不是作用域**‌  
    你在 `const obj = { ... }` 内部定义箭头函数 `b` 时，整个对象字面量是写在‌**全局作用域**‌下的，对象本身不会创建新的函数作用域。  
    此时箭头函数 `b` 能找到的最近一层非箭头函数的外层作用域，就是全局作用域，所以它捕获的 `this` 就是全局对象（浏览器中为 `window`）。
    
3. ‌**严格模式的影响**‌  
    代码开头的 `"use strict"` 不会改变箭头函数的词法绑定逻辑，它只是让普通函数在全局调用时 `this` 变为 `undefined`，但这里箭头函数直接捕获了全局作用域的 `this`，所以最终输出 `window`。