
在 TypeScript 中，定义方法（函数）类型主要有以下几种方式。根据使用场景的不同（如定义变量、接口属性、类成员等），可以选择最适合的一种。

  
## 一. 普通用法
### 1. 函数声明式 (Function Declaration)

这是最传统的定义方式，直接通过 `function` 关键字定义，TypeScript 会自动推断或允许显式指定参数和返回值类型。

```typescript

// 显式指定参数类型和返回值类型
function add(x: number, y: number): number {
  return x + y;
}

  

// 无返回值
function log(message: string): void {
  console.log(message);
}

```
  

### 2. 函数表达式 / Lambda 箭头函数 (Function Expression / Arrow Function)

将函数赋值给变量时，需要定义该变量的类型。有两种写法：

#### A. 类型注解写在变量后

```typescript
// 语法: let 变量名: (参数: 类型) => 返回值类型 = 函数实现
const multiply: (x: number, y: number) => number = function(x, y) {
  return x * y;
};

// 箭头函数写法
const divide: (x: number, y: number) => number = (x, y) => x / y;
```


#### B. 使用 `type` 别名定义函数签名（推荐复用）

先定义函数类型，再使用它，代码更清晰。

```typescript
type MathFunc = (x: number, y: number) => number;

const add: MathFunc = (a, b) => a + b;
const sub: MathFunc = (a, b) => a - b;
```

### 3. 接口定义函数类型 (Interface with Call Signature)

接口不仅可以定义对象结构，还可以定义“可调用”的类型。这在定义回调函数或插件系统时非常有用。

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}
  
// 实现该接口
const mySearch: SearchFunc = function(src, sub) {
  return src.search(sub) !== -1;
};
```

### 4. 类型别名定义对象中的方法 (Type Alias for Object Methods)

当方法是对象的一部分时，可以使用类型别名或接口来定义对象结构。

```typescript
// 方式一：直接在对象类型中定义
type User = {
  name: string;
  greet: (msg: string) => void; // 箭头函数风格的方法类型
};


// 方式二：使用传统方法签名风格（在 interface 中更常见，但 type 也支持类似结构配合索引签名等，通常推荐 interface 定义对象方法）
interface Person {
  name: string;
  sayHi(name: string): string; // 传统方法签名
}

  
const user: Person = {
  name: "Alice",
  sayHi(name: string) {
    return `Hello, ${name}`;
  }
};
```

### 5. 可选参数、默认参数与剩余参数

TypeScript 支持 ES6+ 的函数特性，并在类型系统中加以约束。

#### A. 可选参数 (`?`)

可选参数必须位于必选参数之后。

```typescript
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return firstName + " " + lastName;
  } else {
    return firstName;
  }
}
```


#### B. 默认参数 (`=`)

带有默认值的参数不需要放在最后，但通常建议放在后面以提高可读性。

```typescript
function buildNameDefault(firstName: string, lastName: string = "Smith"): string {
  return firstName + " " + lastName;
}
```

  

C. 剩余参数 (`...args`)

用于处理不确定数量的参数，类型必须定义为数组。

```typescript
function pushToArray(arr: any[], ...items: any[]): void {
  items.forEach(item => arr.push(item));
}

  
let numbers = [1, 2];
pushToArray(numbers, 3, 4, 5);
```
  

### 6. 函数重载 (Function Overloads)

当同一个函数需要根据不同的参数返回不同类型的值时，可以使用重载。注意：TypeScript 的重载是通过多个函数签名声明和一个实现签名组成的。

```typescript
// 重载签名 1
function padding(all: number): string;

// 重载签名 2
function padding(top: number, right: number, bottom: number, left: number): string;

// 实现签名 (对外不可见，类型需兼容所有重载)
function padding(top: number, right?: number, bottom?: number, left?: number): string {
  if (right === undefined && bottom === undefined && left === undefined) {
    return `${top}px`;
  }
  return `${top}px ${right}px ${bottom}px ${left}px`;
}
```

  


## 二. 高阶用法

### 1.只是告诉Ts编译器存在某类型，但是没有实际使用

``` Ts
export interface JSTypeMap {
    string: string;
    number: number;
    boolean: boolean;
    object: object;
    function: Function;
    symbol: symbol;
    undefined: undefined;
    bigint: bigint;
}

export type JSType = keyof JSTypeMap;

export type ArgsType<T extends JSType[]> = {
    [K in keyof T]: JSTypeMap[T[K]];
};

// 声明函数签名
// 仅仅是一个类型声明，它告诉 TypeScript 编译器：“存在这样一个函数，它的参数和返回值长这样”。
declare function test<T extends JSType[]>(
    ...args: [...T, (...args: ArgsType<T>) => any]
): any;

//* 只是对这个声明的‌调用示例‌，用于验证declare function类型是否正确，同样没有实际逻辑
test('boolean', 'string', 'number', (a, b, c) => a + b + c);

```

### 2. 单体函数类型-实际使用（方式 A：Type  &  方式B：Interface）

#### A.函数签名 

在这个文件中，你只负责描述“它长什么样”，不负责“它做什么”。为了能让其他文件导入这个类型约束，你需要将签名导出。
```ts
// types.ts

export interface JSTypeMap {
    string: string;
    number: number;
    boolean: boolean;
    object: object;
    function: Function;
    symbol: symbol;
    undefined: undefined;
    bigint: bigint;
}

export type JSType = keyof JSTypeMap;

export type ArgsType<T extends JSType[]> = {
    [K in keyof T]: JSTypeMap[T[K]];
};

// 注意：这里不再使用 declare，而是直接导出一个函数类型的常量或接口
// 方式 A：导出一个函数变量类型（推荐，更灵活）
export type TestFunction = <T extends JSType[]>(
    ...args: [...T, (...args: ArgsType<T>) => any]
) => any;

// 或者 方式 B：直接声明并导出一个函数（如果在 .ts 文件中，这通常意味着你要在这里提供默认实现，或者配合 d.ts 使用）
// 但在分离实现的场景下，我们通常只导出类型，或者导出一个带有签名的空函数占位符（不推荐），
// 最好的做法是：只在 .d.ts 或纯类型文件中用 declare，或者在 .ts 文件中导出具体实现。

// 鉴于你要“分开实现”，建议如下结构：
// 1. 如果这是 .d.ts 文件：保留 declare function test... 并 export 它。
// 2. 如果这是 .ts 文件：你应该直接写出实现，或者导出一个接口。

// 让我们假设这是一个纯类型定义文件 (types.ts)，我们只导出类型供实现文件使用：
export interface ITestSignature {
    <T extends JSType[]>(...args: [...T, (...args: ArgsType<T>) => any]): any;
}

```

==**案例**==
 要初始化 `ff`，你需要提供一个函数实现，该实现必须同时兼容接口 `Func` 中定义的两个签名：
1. 接收 `number`，返回 `number`
2. 接收 `string`，返回 `void`

使用==联合类型 + 类型守卫（推荐）==
- 这是最标准、最安全的写法。将参数类型定义为两个签名参数的联合类型 `number | string`，返回值类型定义为两个签名返回值的联合类型 `number | void`。
```ts
interface Func { 
    (x: number): number; 
    (a: string): void; 
}

const ff: Func = (arg: number | string): number | void => {
    if (typeof arg === 'number') {
        // 对应 (x: number): number
        return arg * 2;
    } else {
        // 对应 (a: string): void
        console.log(`Hello, ${arg}`);
        // void 类型不需要返回值，或者显式返回 undefined
    }
};

// 调用测试
const res1 = ff(10);      // res1 类型为 number，值为 20
const res2 = ff("World"); // res2 类型为 void，控制台输出 "Hello, World"

```



#### B.导入并实现 （使用...args： ==传参不够透明==）

在这个文件中，你引入类型约束，并编写真正的 JavaScript/TypeScript 逻辑。由于泛型 `T` 在运行时不存在，你需要通过解析参数来实现逻辑。
```Ts
// impl.ts
import type { JSType, ArgsType, ITestSignature } from './types';

// 实现符合 ITestSignature 签名的函数 (有点像是class类 实现 interface接口)
export const test: ITestSignature = function (...args: any[]) {
    // 1. 分离出最后一个参数（回调函数）
    const callback = args[args.length - 1];
    
    // 2. 分离出前面的类型描述符（如 'boolean', 'string'）
    // 注意：在实际业务中，你可能需要根据这些描述符去做数据校验或转换
    const typeDescriptors = args.slice(0, -1);

    console.log("接收到的类型描述:", typeDescriptors);

    // 3. 执行回调
    // 注意：由于签名中回调函数的参数是由泛型决定的，但在运行时，
    // 这个 test 函数本身并没有接收到那些具体的 a, b, c 值。
    // 【重要提示】：你原始的签名设计可能存在逻辑缺口。
    // 原始签名: test(...T, callback) -> 这意味着 test 只接收类型字符串和回调。
    // 但是 callback 需要 (a, b, c)。这些 a,b,c 从哪里来？
    // 如果 test 的职责是“立即执行”，它缺少数据源。
    // 如果 test 的职责是“返回一个强类型函数”，则签名应改为返回函数。
    
    // 假设你的意图是：test 是一个工厂，返回一个强类型的执行函数？
    // 或者，test 接收数据？
    
    // 如果严格按照你给的签名 `...args: [...T, (...args: ArgsType<T>) => any]`
    // 调用时： test('boolean', 'string', (b, s) => ...)
    // 此时 b 和 s 并没有传入 test！
    
    // 因此，通常这种签名的实现有两种可能：
    // A. 它只是一个类型断言工具，运行时直接返回 callback，让用户自己传参？
    // B. 签名其实应该是： test(...types)(...data)(callback) ?
    
    // 为了演示“如何实现并导出”，我们假设 test 的作用是：
    // 接收类型描述，返回一个包装函数，该包装函数接收数据并执行回调。
    // 但这改变了你的签名。
    
    // 如果必须严格匹配你的签名，且假设这是一个“测试桩”或“类型检查器”：
    if (typeof callback !== 'function') {
        throw new Error("Last argument must be a function");
    }
    
    // 由于无法在没有数据的情况下执行 callback，这里仅做演示：
    // 也许你的本意是 let result = test('string', (s) => s.length); 
    // 这种情况下，test 内部无法调用 callback，除非它返回 callback 给外部调用。
    
    return callback; 
};

```


### 3. 多函数类型

类似于==“**单体函数类型中使用接口声明的方式（interrface）**”==实现的方式, 只是在interface中定义多个函数的函数类型；采用键值对的形式；

#### A. 定义多个函数类型
```Ts
export interface JSTypeMap {
    string: string;
    number: number;
    boolean: boolean;
    object: object;
    function: Function;
    symbol: symbol;
    undefined: undefined;
    bigint: bigint;
}

export type JSType = keyof JSTypeMap;

export type ArgsType<T extends JSType[]> = {
    [K in keyof T]: JSTypeMap[T[K]];
};

// 注意：这里不再使用 declare，而是直接导出一个函数类型的常量或接口
// 方式 A：导出一个函数变量类型（推荐，更灵活）
export type TestFunction = <T extends JSType[]>(
    ...args: [...T, (...args: ArgsType<T>) => any]
) => any;

// 或者 方式 B：直接声明并导出一个函数（如果在 .ts 文件中，这通常意味着你要在这里提供默认实现，或者配合 d.ts 使用）
// 但在分离实现的场景下，我们通常只导出类型，或者导出一个带有签名的空函数占位符（不推荐），
// 最好的做法是：只在 .d.ts 或纯类型文件中用 declare，或者在 .ts 文件中导出具体实现。

// 鉴于你要“分开实现”，建议如下结构：
// 1. 如果这是 .d.ts 文件：保留 declare function test... 并 export 它。
// 2. 如果这是 .ts 文件：你应该直接写出实现，或者导出一个接口。

// 让我们假设这是一个纯类型定义文件 (types.ts)，我们只导出类型供实现文件使用：
export interface ITestSignature {
   testFn: <T extends JSType[]>(...args: [...T, (...args: ArgsType<T>) => any]) => any;
   
   otherFn: (name:string)=>void;
}
```
#### B.使用方式（ITestSignature\['testFn'\]）

```Ts
// 实现符合 ITestSignature 签名的函数 (有点像是class类 实现 interface接口)
export const test: ITestSignature['testFn']= function (...args: any[]) {
	// 1. 分离出最后一个参数（回调函数）
	const callback = args[args.length - 1];

	// 2. 分离出前面的类型描述符（如 'boolean', 'string'）
	// 注意：在实际业务中，你可能需要根据这些描述符去做数据校验或转换
	const typeDescriptors = args.slice(0, -1);

	console.log('接收到的类型描述:', typeDescriptors);

	// 3. 执行回调
	// 注意：由于签名中回调函数的参数是由泛型决定的，但在运行时，
	// 这个 test 函数本身并没有接收到那些具体的 a, b, c 值。
	// 【重要提示】：你原始的签名设计可能存在逻辑缺口。
	// 原始签名: test(...T, callback) -> 这意味着 test 只接收类型字符串和回调。
	// 但是 callback 需要 (a, b, c)。这些 a,b,c 从哪里来？
	// 如果 test 的职责是“立即执行”，它缺少数据源。
	// 如果 test 的职责是“返回一个强类型函数”，则签名应改为返回函数。

	// 假设你的意图是：test 是一个工厂，返回一个强类型的执行函数？
	// 或者，test 接收数据？

	// 如果严格按照你给的签名 `...args: [...T, (...args: ArgsType<T>) => any]`
	// 调用时： test('boolean', 'string', (b, s) => ...)
	// 此时 b 和 s 并没有传入 test！

	// 因此，通常这种签名的实现有两种可能：
	// A. 它只是一个类型断言工具，运行时直接返回 callback，让用户自己传参？
	// B. 签名其实应该是： test(...types)(...data)(callback) ?

	// 为了演示“如何实现并导出”，我们假设 test 的作用是：
	// 接收类型描述，返回一个包装函数，该包装函数接收数据并执行回调。
	// 但这改变了你的签名。

	// 如果必须严格匹配你的签名，且假设这是一个“测试桩”或“类型检查器”：
	if (typeof callback !== 'function') {
		throw new Error('Last argument must be a function');
	}

	// 由于无法在没有数据的情况下执行 callback，这里仅做演示：
	// 也许你的本意是 let result = test('string', (s) => s.length);
	// 这种情况下，test 内部无法调用 callback，除非它返回 callback 给外部调用。

	return callback;
};

// 注意 “ = function () { ...” 中并没有编写参数，但是编辑器报错
export const other: ITestSignature['otherFn'] = function () {
	console.log('other');
};
// 反而调用时候，没有传参，编辑器就会报错
other();
// 总结：const other: ITestSignature['otherFn'] 相当于强行修改了它的类型

```
##  三. 总结与建议

| 场景             | 推荐方式                                                               | 示例                                                                                                                                               |
| -------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| ‌普通工具函数‌       | 函数声明式                                                              | function add(a: number, b: number): number {}                                                                                                    |
| ‌回调函数/变量‌      | 类型别名 + 箭头函数                                                        | type Callback = () => void; const cb: Callback = () => {};                                                                                       |
| ‌对象/类的方法‌      | 接口 (Interface or type)(==结合表格最后==)                                 | interface Greeter {<br>  greet(name: string): void; // 接口风格<br>}<br><br>type Greeter = {<br>  greet: (name: string) => void; // 类型别名风格<br>};<br> |
| ‌==复杂函数结构‌==   | 接口调用签名(==结合表格最后==)                                                 | interface Func { (x: number): number; }                                                                                                          |
| ‌不确定参数个数‌      | 剩余参数                                                               | func(...args: number[])                                                                                                                          |
| declare声明      | 声明类型，常存在于.d.ts类型文件中；就是告诉ts编译器，存在xxx类型；防止提示报错。常用来，在使用没有提供类型的第三方库时候； | declare function test<T extends JSType[]>(<br>	...args: [...T, (...args: ArgsType<T>) => any]<br>): any;                                         |
| 函数重载           | 同名不同参数或返回的function，多次定义                                            |                                                                                                                                                  |
| ==单体函数（接口定义）== | 被interface  Name包裹，包含的都会合成一个函数类型，相当于没有key                          | 使用方式  fn:Name = ()=>{}                      相当于强行修改了它的类型                                                                                         |
| ==多个函数（接口定义）== | 被interface  Name包裹，包含多个函数类型，分别是key：value；的格式；                      | 使用方式 fn:Name\['key'\] = ()=>{}                  相当于强行修改了它的类型                                                                                     |


最佳实践：

1. 优先使用 箭头函数类型定义 (`(args) => ReturnType`) 来描述函数形状，因为它简洁且符合现代 JS 习惯。

2. 如果函数类型需要在多处复用，使用 `type` 或 `interface` 进行提取。

3. 尽量明确指定返回值类型，这有助于编译器检查逻辑错误并提高代码可读性。



<br>参考资料<br>[1] [TypeScript入门笔记(三):函数 - 腾讯云](https://cloud.tencent.com/developer/article/2522000)<br>[2] [TypeScript学习笔记(三) - 方法 - 博客园](https://www.cnblogs.com/niklai/p/5763254.html)<br>[3] [TypeScript中对象类型定义的几种方式 - 腾讯云](https://cloud.tencent.com/developer/article/2442744)<br>[4] [typescript 的一些概念 - 阳小年](http://zhuanlan.zhihu.com/p/696414417)<br>[5] [「Vue3+TS专栏课」在Vue3 中如何运用 Typescript ? - 前端进阶学习站](http://www.bilibili.com/video/BV1ASbbzCE1o)<br>[6] [何时使用TypeScript:常见场景的详细介绍 - 腾讯云](https://cloud.tencent.com/developer/news/639331)<br>[7] [Typescript泛型:从基础到进阶的完整指南 - 百度智能云](https://cloud.baidu.com/article/3671302)<br>[8] [TypeScript 中类的理解及应用场景 - 腾讯云](https://cloud.tencent.com/developer/article/2392885)<br>[9] [TypeScript 学习笔记 - 装饰器 - tiny](https://zhuanlan.zhihu.com/p/477872644)<br>[10] [【全110集】TypeScript从入门到精通教程 - Java基础](http://www.bilibili.com/video/BV1dXopBeEVL?p=15)<br>[11] [带你了解Typescript的14个基础语法 - CSDN技术社区](https://huaweicloud.blog.csdn.net/article/details/121681921)<br>[12] [TypeScript语法(类型注解:、类型断言as、联合类型|、类型守卫typeof、交叉类型&、类型别名type、类型保护is)ts语法 - dontla.blog.csdn.net](https://dontla.blog.csdn.net/article/details/152135267)<br>[13] [TypeScript 基础语法入门指南 - 腾讯云](https://cloud.tencent.com/developer/article/2513930)<br>[14] [【全110集】TypeScript从入门到精通教程 - Java基础](http://www.bilibili.com/video/BV1dXopBeEVL?p=13)<br>[15] [TypeScript 中如何声明函数类型 - 写代码的宝哥](http://www.bilibili.com/video/BV1TV4y1r7yH)<br>[16] [2025年6月版,一天快速精通TypeScript,TS速通教程 - web前端高频面试题](http://www.bilibili.com/video/BV1ThKAzHELK?p=50)<br>[17] [【2025】TypeScript全攻略,一天完全学会 - vue大全](http://www.bilibili.com/video/BV1iy3XzpEbM?p=65)<br>[18] [TypeScript语法(16) - 小黄的鸿蒙工坊](http://www.bilibili.com/video/BV1YVUWBrEAL)<br>[19] [【2025】TypeScript全攻略,一天完全学会 - vue大全](http://www.bilibili.com/video/BV1iy3XzpEbM?p=74)<br>[20] [Azure Functions Node.js 开发者参考 - Microsoft](https://docs.microsoft.com/zh-cn/azure/azure-functions/functions-reference-node/)<br>[21] [TypeScript(四)接口 - CSDN下载](https://download.csdn.net/blog/column/12275958/129345432)<br>[22] [如何在 TypeScript 中定义类型,以及接口、类型别名和类的区别 - 爱咋咋地](https://zhuanlan.zhihu.com/p/684994646)<br>[23] [一文带你了解TypeScript函数类型 - CSDN技术社区](https://cuggz.blog.csdn.net/article/details/110789528)<br>[24] [typescript接口定义多个方法 - 51CTO博客](https://blog.51cto.com/u_16213636/12789519)<br>[25] [typescript 类方法 - 51CTO博客](https://blog.51cto.com/u_16213436/12123539)<br>[26] [人生之外的路途 - 博客园](https://www.cnblogs.com/wangzhaoyv/p/14020443.html)<br>[27] [TypeScript 类型定义实战:常见场景全覆盖 - 51CTO博客](https://blog.51cto.com/u_16739385/14397886)<br>[28] [typescript声明一个请求方法 - 51CTO博客](https://blog.51cto.com/u_16213644/12605860)<br>[29] [typescript 定义抽象方法 typescript 类定义 - 51CTO博客](https://blog.51cto.com/u_16099334/11032294)<br><br>百度AI生成，内容仅供参考