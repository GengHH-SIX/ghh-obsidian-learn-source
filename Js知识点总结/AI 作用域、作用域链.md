# 作用域与作用域链：JavaScript 变量访问的核心规则

## 概述

在 JavaScript 中，“变量为什么能被访问”“为什么有时变量会报错未定义”“为什么内层函数能拿到外层函数的变量”，这些问题的答案都藏在**作用域**与**作用域链**中。它们是 JavaScript 引擎管理变量访问权限的“交通规则”，决定了变量的“活动范围”和“查找路径”。

对于开发者而言，理解作用域与作用域链是写出无命名冲突、无内存泄漏代码的基础，更是掌握闭包、模块化、异步编程的前提；对于面试而言，这两个概念是高频核心考点（如“循环中的闭包问题”“var/let 的作用域差异”），直接区分基础能力与深入理解的层级。

本文将遵循“基础概念→核心规则→进阶原理→实战问题”的教育课程流程，从浅入深拆解知识点，同时覆盖面试高频疑难问题，帮助你建立完整的认知体系。

## 一、基础认知：什么是作用域？（变量的“活动范围”）

作用域的本质是确定==**变量和函数的可访问范围**==，它解决了两个核心问题：

1. **变量隔离**：相同名字的变量在不同范围互不干扰（如“全局的 name”和“函数内的 name”是两个独立变量）；
    
2. **访问控制**：规定“哪些地方能访问这个变量”“哪些地方不能访问”。
    

可以类比生活中的“空间权限”：家里的抽屉（函数作用域）只有家人能打开，小区的公共区域（全局作用域）所有人都能进入，而抽屉里的物品（局部变量），小区外人（外层作用域）无法获取。

### 1.1 作用域的核心规则（入门必背）

- **内层可访问外层**：函数内部（内层作用域）能访问函数外部（外层作用域）的变量；
    
- **外层不可访问内层**：函数外部（外层作用域）无法访问函数内部（内层作用域）的变量；
    
- **同层互不干扰**：同一外层作用域下的两个内层作用域，变量互不影响。
    

代码示例：

```JavaScript
// 全局作用域（外层）
const globalVar = "全局变量";

function outer() {
  // outer 函数作用域（中层）
  const outerVar = "外层变量";

  function inner() {
    // inner 函数作用域（内层）
    const innerVar = "内层变量";
    console.log(innerVar);  // ✅ 内层可访问自身
    console.log(outerVar);  // ✅ 内层可访问外层（outer）
    console.log(globalVar); // ✅ 内层可访问最外层（全局）
  }

  inner();
  console.log(innerVar);  // ❌ 外层（outer）不可访问内层（inner）变量
}

outer();
console.log(outerVar);    // ❌ 全局不可访问 outer 内部变量
```

## 二、作用域的三种类型：从全局到块级

JavaScript 中的==作用域分为三类==，分别对应不同的代码场景，其核心差异在于“创建方式”和“访问范围”。

### 2.1 全局作用域：“公共区域”

全局作用域是**最外层的作用域**，生命周期与页面/程序一致（页面不关闭、程序不结束，全局变量就一直存在）。

#### （1）哪些变量属于全局作用域？

- 最外层直接声明的变量/函数（用 `var`/`let`/`const` 均可）；
    
- 未声明直接赋值的变量（隐式全局变量，**强烈不推荐**，易造成全局污染）；
    
- 浏览器环境中 `window` 对象的属性、Node.js 环境中 `global` 对象的属性（显式挂载到全局对象）。
    

代码示例：

```JavaScript
// 1. 最外层声明的全局变量/函数
const globalConst = "全局 const 变量";
let globalLet = "全局 let 变量";
var globalVar = "全局 var 变量";
function globalFunc() {}

// 2. 未声明直接赋值 → 隐式全局变量（危险！）
function createAccidentalGlobal() {
  accidentalGlobal = "我成了全局变量"; // 没有 let/var/const
}
createAccidentalGlobal();
console.log(accidentalGlobal); // ✅ 能访问，属于全局作用域

// 3. 显式挂载到全局对象（浏览器环境）
window.globalByWindow = "通过 window 挂载的全局变量";
console.log(globalByWindow); // ✅ 能访问
```

#### （2）全局作用域的弊端与最佳实践

- **弊端**：
    
    - 命名冲突：多个脚本/模块定义同名全局变量时，后定义的会覆盖先定义的；
        
    - 内存泄漏：全局变量不会被垃圾回收（除非主动赋值为 `null`），长期占用内存；
        
    - 安全性差：任何代码都能修改全局变量，可能导致不可预期的行为。
        
- **最佳实践**：
    
    - 尽量减少全局变量，用模块化（ES Module、CommonJS）封装代码；
        
    - 必须使用全局变量时，用唯一命名空间（如 `const MyApp = {}; MyApp.name = "我的应用"`）；
        
    - 避免未声明直接赋值的隐式全局变量。
        

### 2.2 函数作用域：“私人空间”

函数作用域是**函数内部的独立作用域**，每次函数调用都会创建一个新的函数作用域（即使是同一个函数，多次调用的作用域也相互独立）。

#### （1）核心特性

- 仅函数内部可访问：函数作用域内的变量/函数，外部无法访问；
    
- 函数参数也属于函数作用域：函数的形参相当于在函数内部声明的变量；
    
- `var` 仅支持函数作用域：`var` 声明的变量无法被块级（`{}`）隔离，只能被函数隔离。
    

代码示例：

```JavaScript
function calculate(a, b) {
  // a、b 是函数参数 → 属于函数作用域
  const sum = a + b; // sum 属于函数作用域
  var product = a * b; // var 声明的变量也属于函数作用域
  
  // 函数内部的嵌套函数，也属于当前函数作用域
  function logResult() {
    console.log(`和：${sum}，积：${product}`);
  }
  logResult();
}

calculate(2, 3); // 输出：和：5，积：6
console.log(sum);    // ❌ 外部无法访问 calculate 内部的 sum
console.log(product); // ❌ 外部无法访问 calculate 内部的 product
```

#### （2）函数作用域的“变量提升”陷阱（`var` 特有）

`var` 声明的变量会发生“变量提升”：变量的声明会被提升到函数作用域的顶部，但赋值不会提升。这会导致“变量未赋值却能访问”的情况，是很多 bug 的根源。

代码示例：

```JavaScript
function testVarHoisting() {
  console.log(hoistedVar); // ✅ 输出 undefined（声明提升了，赋值没提升）
  var hoistedVar = "我被提升了";
  console.log(hoistedVar); // ✅ 输出 "我被提升了"
}
testVarHoisting();
```

### 2.3 块级作用域：ES6 的“精细控制”

块级作用域是**由** **`{}`** **包裹的独立作用域**（如 `if`/`for`/`while` 语句块、单独的 `{}` 块），仅 ES6 及以后支持（通过 `let`/`const` 声明变量）。

它解决了 `var` 无法隔离块级变量的问题（如循环中变量共享导致的 bug），是现代 JavaScript 开发的首选。

#### （1）哪些场景会创建块级作用域？

- 单独的 `{}` 块 + `let`/`const`；
    
- `if`/`else` 语句块 + `let`/`const`；
    
- `for`/`while` 循环块 + `let`/`const`；
    
- `switch` 语句块 + `let`/`const`。
    

代码示例：

```JavaScript
// 1. 单独的 {} 块
{
  let blockVar = "块级变量";
  const blockConst = "块级常量";
  var notBlockVar = "var 不支持块级作用域";
  console.log(blockVar);  // ✅ 块内可访问
  console.log(blockConst); // ✅ 块内可访问
}
console.log(blockVar);    // ❌ 块外不可访问
console.log(blockConst); // ❌ 块外不可访问
console.log(notBlockVar); // ✅ var 声明的变量能访问（无块级隔离）

// 2. for 循环块（经典场景）
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 输出：0、1、2（每次循环创建新块级作用域）
}

// 对比 var （无块级隔离，所有循环共享一个变量）
for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100); // 输出：3、3、3（循环结束后 j 为 3）
}
```

#### （2）块级作用域的“暂时性死区（TDZ）”

`let`/`const` 声明的变量虽然也会发生“声明提升”，但提升后会进入“暂时性死区”：在变量声明语句之前访问变量，会直接报错（而非 `undefined`），这是 `let`/`const` 与 `var` 的核心差异之一，避免了“未赋值却访问”的 bug。

代码示例：

```JavaScript
function testTDZ() {
  console.log(tdzVar); // ❌ 报错：Cannot access 'tdzVar' before initialization
  let tdzVar = "我在 TDZ 中";
}
testTDZ();
```

### 2.4 三种作用域与变量声明方式的关系（面试高频对比）

不同的变量声明方式（`var`/`let`/`const`）在作用域支持上差异巨大，是面试必问点，需重点掌握：

|   |   |   |   |
|---|---|---|---|
|特性|`var`|`let`|`const`|
|支持的作用域|全局/函数作用域|全局/函数/块级作用域|全局/函数/块级作用域|
|变量提升|✅ 声明提升（值为 `undefined`）|✅ 声明提升（进入 TDZ）|✅ 声明提升（进入 TDZ）|
|暂时性死区（TDZ）|❌ 无|✅ 有|✅ 有|
|重复声明|✅ 允许（覆盖前值）|❌ 禁止（同作用域报错）|❌ 禁止（同作用域报错）|
|挂载全局对象|✅ 是（如 `window.varVar`）|❌ 否|❌ 否|
|重新赋值|✅ 允许|✅ 允许|❌ 禁止（常量）|
|声明时初始化要求|❌ 无|❌ 无|✅ 必须（否则报错）|

补充：`const` 的“不变”真相 

`const` 声明的“常量”并非“值不变”，而是“变量指向的内存地址不变”：

- 对于原始值（数字、字符串、布尔值）：值存在变量指向的地址中，因此无法修改；
    
- 对于引用值（对象、数组、函数）：变量指向的是“对象的内存地址”，因此可以修改对象的属性/数组的元素，但不能重新赋值变量（指向新对象）。
    

代码示例：

```JavaScript
const constNum = 10;
constNum = 20; // ❌ 报错：不能重新赋值原始值

const constArr = [1, 2];
constArr.push(3); // ✅ 可以修改数组元素（地址未变）
console.log(constArr); // [1, 2, 3]
constArr = [4, 5]; // ❌ 报错：不能重新赋值（指向新数组）

const constObj = { name: "张三" };
constObj.age = 20; // ✅ 可以修改对象属性
console.log(constObj); // { name: "张三", age: 20 }
constObj = { name: "李四" }; // ❌ 报错：不能重新赋值
```

## 三、作用域链：变量查找的“寻宝路线图”

当 JavaScript 引擎需要访问一个变量时，不会“随机查找”，而是遵循一条固定的“查找路径”——这就是**作用域链**。

### 3.1 作用域链的本质

作用域链是**当前作用域及其所有外层作用域组成的链式结构**，每个作用域都有一个“指针”指向它的外层作用域，最终指向全局作用域。

可以类比“寻宝”：你（当前作用域）要找一件宝物（变量），先在自己的抽屉（当前作用域）找；找不到就去家里的柜子（外层作用域）找；再找不到就去小区的储物间（更外层作用域）找；最后去城市的博物馆（全局作用域）找；如果博物馆也没有，就确定“宝物不存在”（报错）。

### ==3.2 作用域链的创建规则（核心原理）==

==作用域链的结构在**函数定义时就已确定**（而非函数调用时），这是因为 JavaScript 采用“词法作用域（静态作用域）”——作用域由代码的书写位置决定，与运行时调用位置无关。==

具体创建流程：

1. 函数定义时，JavaScript 引擎会记录函数“定义时所处的外层作用域”；
    
2. 函数调用时，会创建一个“执行上下文”，其中包含作用域链；
    
3. 作用域链的第一个元素是“当前函数的变量对象”（存储当前作用域的变量/参数）；
    
4. 后续元素依次是“外层作用域的变量对象”，直到全局作用域的“全局对象”。
    

代码示例：

```JavaScript
// 全局作用域（变量对象：globalVar、outer）
const globalVar = "全局变量";

function outer() {
  // outer 函数作用域（变量对象：outerVar、inner）
  const outerVar = "外层变量";

  function inner() {
    // inner 函数作用域（变量对象：innerVar）
    const innerVar = "内层变量";
    console.log(globalVar); // 查找路径：inner → outer → 全局
  }

  return inner; // 函数定义时已记录外层作用域（outer）
}

// 调用 outer，获取 inner 函数（此时 inner 未调用）
const innerFunc = outer();
// 调用 inner（即使在全局作用域调用，作用域链仍为 inner → outer → 全局）
innerFunc(); // ✅ 能访问 globalVar 和 outerVar
```

### 3.3 变量查找的优先级规则（面试易错点）

作用域链的查找遵循“**就近原则**”：

1. 优先查找当前作用域的变量，找到后立即返回，不再向上查找；
    
2. 若当前作用域未找到，沿作用域链向上层作用域查找，直到全局作用域；
    
3. 若全局作用域仍未找到，抛出 `ReferenceError: xxx is not defined`。
    

**易错点**：变量名冲突时，“内层变量会覆盖外层变量”（就近原则），而非报错。

代码示例：

```JavaScript
const name = "全局 name"; // 全局作用域的 name

function outer() {
  const name = "outer name"; // outer 作用域的 name（覆盖全局）

  function inner() {
    const name = "inner name"; // inner 作用域的 name（覆盖 outer）
    console.log(name); // ✅ 输出 "inner name"（当前作用域优先）
  }

  inner();
  console.log(name); // ✅ 输出 "outer name"（outer 作用域优先）
}

outer();
console.log(name); // ✅ 输出 "全局 name"（全局作用域）
```

## 四、进阶：作用域链与闭包的关系（面试核心）

闭包是 JavaScript 中最强大也最易混淆的特性，而它的实现完全依赖于作用域链——==**闭包就是“函数与其定义时的作用域链的组合”**==，即使函数在其他作用域调用，仍能通过作用域链访问定义时的外层变量。

### 4.1 闭包的本质（结合作用域链理解）

当一个内层函数被外层函数“返回”并在外部调用时：

1. 内层函数的作用域链中包含外层函数的变量对象；
    
2. 即使外层函数已执行完毕（理论上其变量对象应被垃圾回收），但由于内层函数仍引用该变量对象，因此它不会被回收；
    
3. 这就导致内层函数能“记住”定义时的外层变量，形成闭包。
    

代码示例：

```JavaScript
function createCounter() {
  let count = 0; // outer 函数的变量（应被回收，但被闭包保留）

  // 内层函数（闭包）：作用域链包含 createCounter 的变量对象
  return function increment() {
    count++;
    return count;
  };
}

// 调用 createCounter，获取 increment 函数（闭包）
const counter = createCounter();
// 此时 createCounter 已执行完毕，但 count 被闭包保留
console.log(counter()); // 1
console.log(counter()); // 2（count 持续累加，未被回收）
console.log(counter()); // 3
```

### 4.2 闭包的常见应用场景

==闭包的核心价值是“**保留外层作用域的变量**”，常见应用包括：==

1. **创建私有变量**：通过闭包隐藏变量，仅暴露操作接口（模块化的基础）；
    
2. **函数柯里化**：将多参数函数转为单参数函数，实现参数复用；
    
3. **防抖/节流函数**：保留定时器 ID 和触发状态，避免重复执行；
    
4. **循环中保存变量**：解决 `var` 循环变量共享的问题（ES6 前常用，现在被 `let` 替代）。
    

代码示例：创建私有变量（模块化）

```JavaScript
function createUser(name) {
  // 私有变量：仅能通过闭包访问，外部无法直接修改
  let age = 18;

  // 暴露操作接口（闭包）
  return {
    getName: () => name, // 读取 name
    getAge: () => age,   // 读取 age
    setAge: (newAge) => { // 修改 age（可加校验逻辑）
      if (newAge >= 0 && newAge <= 150) {
        age = newAge;
      } else {
        throw new Error("年龄必须在 0-150 之间");
      }
    }
  };
}

const user = createUser("张三");
console.log(user.getName()); // "张三"
console.log(user.getAge());  // 18
user.setAge(20);
console.log(user.getAge());  // 20
user.age = 30; // ❌ 无法直接修改私有变量（外部无访问权限）
console.log(user.getAge());  // 20（未被修改）
```

## 五、面试高频疑难问题解析（附答案）

作用域与作用域链是面试必问点，以下是高频问题及深度解析，帮助你应对面试。

### 5.1 问题1：为什么 `for(var i=0; i<3; i++) { setTimeout(...)}` 输出 3 个 3？如何解决？

#### 问题分析：

- `var` 仅支持函数作用域，不支持块级作用域，因此 `for` 循环中的 `i` 属于全局作用域（或外层函数作用域）；
    
- 循环执行时，`setTimeout` 是异步函数，会在循环结束后执行；
    
- 循环结束后，`i` 的值已变为 3，因此 3 个 `setTimeout` 都输出 3。
    

#### 解决方法：

- 方法1：用 `let` 声明 `i`（`let` 支持块级作用域，每次循环创建新的 `i`）；
    
- 方法2：用闭包保留每次循环的 `i`（ES6 前的解决方案）。
    

代码示例：

```JavaScript
// 问题代码
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 输出：3、3、3
}

// 解决方法1：用 let（推荐）
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 输出：0、1、2
}

// 解决方法2：用闭包（ES6 前）
for (var j = 0; j < 3; j++) {
  // 立即执行函数（IIFE）创建闭包，保留每次循环的 j
  (function (currentJ) {
    setTimeout(() => console.log(currentJ), 100);
  })(j); // 传递当前循环的 j
}
// 输出：0、1、2
```

### 5.2 问题2：`var`/`let`/`const` 在全局作用域下的差异？（如是否挂载到 `window`）

#### 答案：

- `var` 声明的全局变量：会挂载到全局对象（浏览器 `window`、Node.js `global`），可通过 `window.varVar` 访问；
    
- ==`let`/`const` 声明的全局变量：不会挂载到全局对象，仅存在于全局作用域中，无法通过 `window` 访问==；
    
- 其他差异：`var` 允许重复声明，`let`/`const` 禁止；`var` 有变量提升且无 TDZ，`let`/`const` 有 TDZ。
    

代码示例（浏览器环境）：

```JavaScript
var varGlobal = "var 全局变量";
let letGlobal = "let 全局变量";
const constGlobal = "const 全局变量";

console.log(window.varGlobal);    // ✅ "var 全局变量"（挂载到 window）
console.log(window.letGlobal);    // ❌ undefined（不挂载）
console.log(window.constGlobal);   // ❌ undefined（不挂载）

// 重复声明差异
var varGlobal = "var 重复声明"; // ✅ 允许，覆盖前值
let letGlobal = "let 重复声明"; // ❌ 报错：Identifier 'letGlobal' has already been declared
```

### 5.3 问题3：什么是==暂时性死区（TDZ）==？它有什么作用？

#### 答案：

- **定义**：暂时性死区是 `let`/`const` 特有的特性——变量声明被提升到作用域顶部后，从作用域开始到变量声明语句之间的区域，称为“暂时性死区”。在 TDZ 中访问变量，会直接抛出 `ReferenceError`，而非 `undefined`。
    
- **作用**：
    
    - 避免“变量未赋值却访问”的 bug（`var` 的变量提升常导致此问题）；
        
    - 强制开发者“先声明、后使用”变量，提升代码可读性和规范性。
        

代码示例：

```JavaScript
function testTDZ() {
  // 从函数开始到 let 声明之间，是 tdzVar 的 TDZ
  console.log(tdzVar); // ❌ 报错：Cannot access 'tdzVar' before initialization
  let tdzVar = "走出 TDZ 了";
  console.log(tdzVar); // ✅ 输出 "走出 TDZ 了"
}
testTDZ();
```

### 5.4 问题4：作用域链与原型链的区别？（面试高频对比）

#### 答案：

作用域链和原型链是 JavaScript 中两个完全不同的“链结构”，核心差异如下：

| 维度   | 作用域链                | 原型链                                 |
| ---- | ------------------- | ----------------------------------- |
| 作用对象 | 变量和函数的访问            | 对象属性的访问                             |
| 形成原因 | 函数嵌套定义（词法作用域）       | 对象继承（`prototype` 属性）                |
| 查找方向 | 从内到外（当前作用域 → 全局作用域） | 从下到上（子对象 → 原型对象 → Object.prototype） |
| 查找内容 | 变量名、函数名             | 对象的属性/方法                            |
| 失败结果 | 抛出 `ReferenceError` | 返回 `undefined`（非严格模式）               |

代码示例对比：

```JavaScript
// 1. 作用域链（变量查找）
const name = "全局 name";
function outer() {
  function inner() {
    console.log(name); // 作用域链查找：inner → outer → 全局
  }
  inner();
}
outer(); // ✅ 输出 "全局 name"

// 2. 原型链（对象属性查找）
const parent = { name: "parent name" };
const child = Object.create(parent); // child 的原型是 parent
console.log(child.name); // 原型链查找：child → parent → Object.prototype
// ✅ 输出 "parent name"（child 自身无 name，从原型 parent 查找）
console.log(child.age); // ❌ 输出 undefined（原型链全程未找到 age）
```

## 六、总结与最佳实践

### 6.1 核心知识点总结

1. **作用域**：变量的可访问范围，分为全局、函数、块级三类，核心规则是“内层可访问外层，外层不可访问内层”；
    
2. **变量声明**：`let`/`const` 支持块级作用域，有 TDZ，禁止重复声明；`var` 仅支持函数作用域，有变量提升，无 TDZ；
    
3. **作用域链**：变量查找的路径，由函数定义时的词法环境决定，遵循“就近原则”；
    
4. **闭包**：函数与其定义时的作用域链的组合，能保留外层变量，依赖作用域链实现；
    
5. **关键原则**：JavaScript 采用词法作用域，作用域链在函数定义时确定，与调用位置无关。
    

### 6.2 开发最佳实践

1. **优先使用** **`let`****/****`const`**：避免 `var` 的变量提升和全局污染，`const` 优先（非必要不重新赋值）；
    
2. **减少全局变量**：用模块化（ES Module）或 IIFE（立即执行函数）封装代码，避免全局污染；
    
3. **谨慎使用闭包**：闭包会导致外层变量无法回收，避免不必要的闭包；使用后及时解除引用（如 `counter = null`）；
    
4. **避免未声明赋值**：禁止隐式全局变量（未声明直接赋值），防止命名冲突；
    
5. **调试技巧**：用 Chrome DevTools 的“Sources”面板设置断点，观察“Scope”面板中的作用域链（包含 `Local`、`Closure`、`Global`）。
    

通过掌握作用域与作用域链，你能从根本上理解变量的访问规则，避免大部分“变量未定义”“变量被覆盖”的 bug，同时为后续学习闭包、模块化、异步编程打下坚实基础。