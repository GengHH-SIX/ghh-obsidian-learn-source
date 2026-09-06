

## 概述

JavaScript 中的运算符、表达式和类型转换是构建代码逻辑的基础，三者紧密关联：运算符定义了值之间的计算规则，表达式通过运算符组合值产生结果，而类型转换则解决了不同类型值参与运算时的兼容性问题。

JavaScript 是动态类型语言，变量类型无需预先声明，这使得类型转换成为高频场景，也容易因规则理解不透彻导致意外结果。本文将系统梳理运算符的核心规则、表达式的执行逻辑，以及显式/隐式类型转换的细节，帮助开发者精准掌控代码行为。

## 一、运算符规则

JavaScript 运算符按功能可分为**算术运算符**、**比较运算符**、**逻辑运算符**、**赋值运算符**等，不同类型运算符遵循特定的优先级、结合性和运算规则。

### 1. 运算符优先级与结合性

- **优先级**：决定多个运算符共存时的执行顺序，优先级高的先执行（如乘法 `*` 高于加法 `+`）。
    
- **结合性**：当多个同优先级运算符连续出现时，决定执行方向（多数为左结合，即从左到右；赋值运算符、三元运算符为右结合）。
    

核心优先级排序（从高到低，仅列常用）：

1. 圆括号 `()`（改变优先级）
    
2. 一元运算符（`++`、`--`、`!`、`typeof` 等）
    
3. 算术运算符（`*`、`/`、`%` 高于 `+`、`-`）
    
4. 比较运算符（`>`、`<`、`>=`、`<=`、`==`/`===`、`!=`/`!==`）
    
5. 逻辑运算符（`&&` 高于 `||`）
    
6. 三元运算符 `? :`
    
7. 赋值运算符（`=`、`+=`、`-=` 等）
    

结合性示例：

```JavaScript
// 左结合：3 + 4 + 5 等价于 (3 + 4) + 5
console.log(3 + 4 + 5); // 12

// 右结合：a = b = 5 等价于 a = (b = 5)
let a, b;
a = b = 5;
console.log(a, b); // 5 5
```

### 2. 常用运算符核心规则

#### （1）算术运算符

包括 `+`、`-`、`*`、`/`、`%`、`++`、`--`，核心规则如下：

- 运算数非数字类型时，会隐式转换为数字（`NaN` 参与运算结果均为 `NaN`）。
    
- `+` 运算符：若有一个运算数是字符串，执行字符串拼接；否则执行数字加法。
    
- `%` 运算符：结果符号与被除数一致（如 `7 % 3 = 1`，`-7 % 3 = -1`）。
    
- `++`/`--`：前置先自增/自减再参与运算，后置先参与运算再自增/自减。
    

示例：

```JavaScript
console.log(1 + "2"); // "12"（字符串拼接）
console.log(1 + true); // 2（true 转为 1）
console.log(7 % 3); // 1
console.log(-7 % 3); // -1
let x = 2;
console.log(x++ + 3); // 5（先运算再自增，x 最终为 3）
console.log(++x + 3); // 7（先自增再运算，x 最终为 4）
```

#### （2）比较运算符

包括 `==`（相等）、`===`（严格相等）、`>`、`<` 等，核心规则如下：

- `===`：严格匹配，值和类型必须完全一致（无类型转换）。
    
- `==`：宽松匹配，会先进行类型转换再比较（仅值相等即可）。
    
- 字符串比较：按 Unicode 编码顺序逐字符对比。
    
- 对象比较：比较引用地址，仅当指向同一对象时返回 `true`。
    

示例：

```JavaScript
console.log(1 == "1"); // true（"1" 转为 1）
console.log(1 === "1"); // false（类型不同）
console.log(null == undefined); // true（特殊规则）
console.log(null === undefined); // false（类型不同）
console.log("a" > "b"); // false（"a" 的 Unicode 编码小于 "b"）
console.log({} === {}); // false（两个不同对象，引用不同）
```

#### （3）逻辑运算符

包括 `&&`（逻辑与）、`||`（逻辑或）、`!`（逻辑非），遵循“短路求值”规则：

- `&&`：左侧为 `false` 时，直接返回左侧值，不执行右侧；左侧为 `true` 时，返回右侧值。
    
- `||`：左侧为 `true` 时，直接返回左侧值，不执行右侧；左侧为 `false` 时，返回右侧值。
    
- `!`：将运算数转为布尔值后取反（双重 `!!` 可快速转为布尔值）。
    

示例：

```JavaScript
// 短路求值：右侧 console.log 不执行
false && console.log("逻辑与右侧");
true || console.log("逻辑或右侧");

// 返回实际值（非布尔值）
console.log(2 && 3); // 3（左侧为真，返回右侧）
console.log(0 || "默认值"); // "默认值"（左侧为假，返回右侧）

// 双重 ! 转为布尔值
console.log(!!"hello"); // true
console.log(!!0); // false
```

#### （4）赋值运算符

包括 `=`、`+=`、`-=`、`*=` 等，核心规则：

- 右结合性，先计算右侧表达式的值，再赋值给左侧变量。
    
- 复合赋值运算符（如 `+=`）会先执行运算再赋值，且不会改变变量类型（如字符串 `+=` 为拼接）。
    

示例：

```JavaScript
let num = 10;
num += 5; // 等价于 num = num + 5 → 15
let str = "hello";
str += " world"; // 等价于 str = str + " world" → "hello world"
```

## 二、表达式

表达式是由运算符、变量、字面量等组合而成的代码片段，其核心特征是“能产生一个值”。根据运算符类型，常见表达式可分为以下几类：

### 1. 算术表达式

由算术运算符组合而成，结果为数字或 `NaN`（运算失败时）。

```JavaScript
const sum = 3 + 4 * 2; // 11（遵循优先级）
const remainder = 10 % 3; // 1
```

### 2. 比较表达式

由比较运算符组合而成，结果恒为布尔值（`true`/`false`）。

```JavaScript
const isGreater = 5 > 3; // true
const isEqual = "5" == 5; // true（宽松比较）
```

### 3. 逻辑表达式

由逻辑运算符组合而成，结果为“短路求值”过程中最后一个执行的表达式的值（不一定是布尔值）。

```JavaScript
const result1 = "a" && 123; // 123（左侧为真，返回右侧）
const result2 = 0 || null || "default"; // "default"（前两个为假，返回最后一个）
```

### 4. 赋值表达式

由赋值运算符组合而成，结果为赋值后变量的值（支持链式赋值）。

```JavaScript
const x = 10; // 表达式结果为 10
const y = x += 5; // 等价于 x = x + 5，y 最终为 15
```

### 5. 三元表达式

格式：`条件表达式 ? 表达式1 : 表达式2`，右结合性。

- 条件为 `true` 时执行表达式1，否则执行表达式2，结果为对应表达式的值。
    

示例：

```JavaScript
const age = 18;
const status = age >= 18 ? "成年" : "未成年"; // "成年"
```

### 表达式执行顺序

- 遵循运算符优先级和结合性。
    
- 括号 `()` 可强制改变执行顺序，优先级最高。
    
- 函数调用、属性访问（`.`、`[]`）等操作的优先级高于算术运算符，需注意组合场景：
    
    ```JavaScript
    const obj = { a: 2 };
    const result = obj.a + 3 * 2; // 2 + 6 = 8（属性访问优先级高于乘法）
    ```
    

## 三、类型转换

JavaScript 类型转换分为**显式转换**（开发者主动调用方法转换）和**隐式转换**（引擎自动触发，多发生在运算或比较时），核心转换目标为三种基本类型：字符串、数字、布尔值。

### 1. 显式类型转换

开发者通过内置方法主动转换类型，规则明确，不易出错。

#### （1）转为字符串：`String()` 或 `toString()`

- `String()`：可处理所有类型（包括 `null` 和 `undefined`）。
    
- `toString()`：不支持 `null` 和 `undefined`（调用会报错），数字类型可传参数指定进制。
    

示例：

```JavaScript
// String() 方法
console.log(String(123)); // "123"
console.log(String(null)); // "null"
console.log(String(undefined)); // "undefined"
console.log(String(true)); // "true"

// toString() 方法
console.log((123).toString()); // "123"
console.log((123).toString(2)); // "1111011"（二进制）
// console.log(null.toString()); // 报错：Cannot read property 'toString' of null
```

#### （2）转为数字：`Number()`、`parseInt()`、`parseFloat()`

- `Number()`：严格转换，处理整个值，转换失败返回 `NaN`。
    
- `parseInt()`：从左到右解析整数，遇到非数字字符停止，支持指定进制，默认十进制。
    
- `parseFloat()`：解析浮点数（支持一个小数点），遇到非数字字符停止。
    

转换规则（`Number()`）：

|   |   |
|---|---|
|输入类型|转换结果|
|字符串|纯数字字符串转为对应数字，否则 `NaN`|
|布尔值|`true` 转 1，`false` 转 0|
|`null`|0|
|`undefined`|`NaN`|
|对象|先调用 `valueOf()`，再尝试转换；若无则调用 `toString()`|

示例：

```JavaScript
console.log(Number("123")); // 123
console.log(Number("123abc")); // NaN
console.log(Number(true)); // 1
console.log(Number(null)); // 0

console.log(parseInt("123abc")); // 123
console.log(parseInt("12.34")); // 12
console.log(parseFloat("12.34abc")); // 12.34
```

#### （3）转为布尔值：`Boolean()` 或双重 `!`

转换规则：只有以下 6 种“假值”转为 `false`，其余均为 `true`：

- `undefined`、`null`、`0`、`NaN`、`""`（空字符串）、`false`
    

示例：

```JavaScript
console.log(Boolean(0)); // false
console.log(Boolean("")); // false
console.log(Boolean(null)); // false
console.log(Boolean(" ")); // true（空格字符串非空）
console.log(Boolean({})); // true（对象恒为真）
console.log(!!123); // true（双重 ! 等价于 Boolean()）
```

### 2. 隐式类型转换

引擎自动触发的转换，多发生在运算符运算、比较判断等场景，其规则与显式转换一致，只是触发时机隐含。

#### （1）算术运算中的隐式转换

- 加法 `+`：若有一个运算数是字符串，触发字符串拼接（其他类型转为字符串）；否则转为数字运算。
    
- 其他算术运算符（`-`、`*`、`/`、`%`）：一律转为数字运算，转换失败结果为 `NaN`。
    

示例：

```JavaScript
// 加法：字符串拼接
console.log(123 + "456"); // "123456"
console.log(true + "abc"); // "trueabc"

// 减法：转为数字
console.log("123" - 45); // 78
console.log("abc" - 10); // NaN
console.log(null - 1); // -1（null 转 0）
console.log(undefined - 1); // NaN（undefined 转 NaN）
```

#### （2）比较运算中的隐式转换

- `==`（宽松相等）：先统一类型再比较，核心规则：
    
    - 若类型相同，直接比较值（同 `===`）。
        
    - 若一个是字符串、一个是数字，字符串转为数字。
        
    - 若一个是布尔值，布尔值转为数字。
        
    - `null` 和 `undefined` 互相相等，但不与其他值相等。
        
    - 若涉及对象，先转为原始类型（`valueOf()` → `toString()`）。
        
- `===`（严格相等）：不触发隐式转换，类型和值均一致才返回 `true`。
    

示例：

```JavaScript
console.log("5" == 5); // true（字符串转数字）
console.log(true == 1); // true（布尔转数字）
console.log(null == undefined); // true（特殊规则）
console.log({} == "[object Object]"); // true（对象转字符串）
console.log("5" === 5); // false（类型不同）
```

#### （3）逻辑运算中的隐式转换

逻辑运算符（`&&`、`||`、`!`）会将运算数隐式转为布尔值，再执行逻辑判断，但返回的是原始值（非布尔值）。

示例：

```JavaScript
// ! 转为布尔值后取反
console.log(!0); // true
console.log(!{}); // false

// && 和 || 短路求值，返回原始值
console.log("hello" && 123); // 123（左侧为真，返回右侧原始值）
console.log(0 || "default"); // "default"（左侧为假，返回右侧原始值）
```

### 3. 类型转换的常见陷阱

- `NaN` 与任何值比较（包括自身）均为 `false`，判断需用 `isNaN()` 或 `Number.isNaN()`。
    
- 空数组 `[]` 转为数字为 0，转为布尔值为 `true`；空对象 `{}` 转为数字为 `NaN`，转为布尔值为 `true`。
    
- `parseInt()` 解析带前导零的字符串时，不会自动视为八进制（ES5 后），但需注意进制参数缺失的问题。
    

示例：

```JavaScript
console.log(NaN == NaN); // false
console.log(Number([])); // 0
console.log(Boolean([])); // true
console.log(parseInt("012")); // 12（ES5 后默认十进制）
console.log(parseInt("0x12")); // 18（自动识别十六进制）
```

## 四、总结

1. 运算符遵循“优先级高于结合性”的执行规则，复杂表达式可通过括号明确执行顺序。
    
2. 表达式的核心是“产生值”，不同类型表达式的结果类型由运算符规则决定（如比较表达式恒为布尔值）。
    
3. 类型转换的核心是“转为字符串、数字、布尔值”，显式转换（`String()`、`Number()` 等）规则明确，隐式转换需关注运算场景（加法可能拼接字符串，其他算术运算转为数字）。
    
4. 开发中推荐使用 `===` 避免隐式类型转换带来的意外，复杂类型转换优先使用显式方法，确保代码可读性和稳定性。
    

要不要我帮你整理一份**类型转换核心规则速查表**，方便快速查阅和开发复用？