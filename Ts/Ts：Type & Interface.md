在 TypeScript 中，`interface`（接口）和 `type`（类型别名）都是用于定义类型结构的核心工具。它们在大多数描述对象形状的场景下可以互换使用，但在扩展性、适用场景和底层机制上存在显著差异。


以下是两者的详细对比：

### 一、核心相同点

#### 1.  描述对象结构

    两者都可以用来定义对象的属性及其类型。
```typescript
// 使用 interface
interface User {
  name: string;
  age: number;
}

// 使用 type
type User = {
  name: string;
  age: number;
};
```

#### 2.  支持继承/扩展
   `interface` 可以通过 `extends` 继承其他接口。
    `type` 可以通过交叉类型（`&`）实现类似继承的效果。
    
```typescript

interface Animal {
  name: string;
}
interface Bear extends Animal {
  honey: boolean;
}


type Animal = {
  name: string;
}
type Bear = Animal & { 
  honey: boolean; 
}

```

#### 3.  类实现（Implements）

    类可以使用 `implements` 关键字来实现 `interface`。虽然类也可以实现 `type` 定义的对象结构，但 `interface` 在此场景下是更标准的选择。

---

### 二、核心不同点

#### 1. 扩展性与合并机制（==最关键的区别==）

*   Interface：支持声明合并（Declaration Merging）

    如果你定义了多个同名的 `interface`，TypeScript 会自动将它们合并为一个接口。这对于扩展第三方库的类型或逐步构建复杂类型非常有用。

    ```typescript
    interface Box {
      height: number;
    }

    interface Box {
      width: number;
    }
    // 最终 Box 等同于 { height: number; width: number; }
    ```


*   Type：不支持合并，重名会报错

    `type` 一旦定义，就不能再次定义同名类型。如果需要扩展，必须创建一个新的类型别名。

    ```typescript
    type Box = {
      height: number;
    }
    // type Box = { width: number; } // ❌ 错误：Duplicate identifier 'Box'
    ```

  
#### 2. 适用的类型范围

*   Interface：主要用于对象和函数签名

    `interface` 的设计初衷是描述对象的形状（Shape）。它可以很好地描述对象属性和方法，但不能直接表示联合类型、元组或原始类型。

    ```typescript
    interface Point {
      x: number;
      y: number;
    }
    // interface StringOrNumber = string | number; // ❌ 错误
    ```

*   Type：通用性更强，支持所有类型

    `type` 是一个类型别名，它可以指向任何类型，包括：

    *   联合类型（Union Types）：`type ID = string | number;`
    *   元组（Tuples）：`type Point = [number, number];`
    *   基本类型别名：`type Name = string;`
    *   映射类型与条件类型：`type Readonly<T> = { readonly [P in keyof T]: T[P] };`

#### 3. 语法差异

*   定义方法：
    *   `interface` 可以直接定义方法签名，无需箭头函数语法。
    *   `type` 定义对象中的方法时，通常使用箭头函数形式或对象字面量方法简写。

    ```typescript
    interface Greeter {
      greet(name: string): void; // 接口风格
    }

    type Greeter = {
      greet: (name: string) => void; // 类型别名风格
    };
    ```

*   计算属性：

    *   `type` 支持更复杂的计算和映射操作。
    *   `interface` 相对静态。

#### 4. 错误提示信息

*   Interface：当类型检查失败时，报错信息通常更清晰、更易读，因为它明确指出了是哪个接口契约被违反。

*   Type：在处理复杂的交叉类型或联合类型时，报错信息可能会比较冗长且难以解读（例如显示为复杂的交集结构）。


---

### 三、选择建议：该用哪个？
  

| 场景             | 推荐选择                 | 原因                                      |               |
| -------------- | -------------------- | --------------------------------------- | ------------- |
| ‌定义对象结构‌       | ‌Interface‌          | 语义更清晰，支持合并，适合面向对象编程。                    |               |
| ‌类实现契约‌        | ‌Interface‌          | implements 关键字与 interface 搭配是 TS 的标准范式。 |               |
| ‌联合/交叉类型‌      | ‌Type‌               | interface 无法直接表示 string                 | number 等联合类型。 |
| ‌元组类型‌         | ‌Type‌               | interface 不支持元组。                        |               |
| ‌工具类型/映射‌      | ‌Type‌               | 需要配合 keyof、in、条件类型等高级特性时。               |               |
| ‌公共 API / 库开发‌ | ‌Interface‌          | 允许用户通过声明合并来扩展你的库类型，灵活性更高。               |               |
| ‌函数类型定义‌       | ‌Type‌ 或 ‌Interface‌ | 两者皆可，但 type 在定义复杂函数签名（如重载）时有时更简洁。       |               |

### 四、总结

*   优先使用 `interface`：
	当你主要是在描述对象的结构，或者希望类型能够被扩展/合并时。它是构建大型应用和库的首选。

*   优先使用 `type`：
	当你需要定义联合类型、元组、函数类型，或者需要进行复杂的类型运算（如映射、条件判断）时。

  

在实际项目中，两者经常混合使用。例如，用 `interface` 定义核心数据模型，用 `type` 定义这些模型的组合状态或工具类型。保持团队内部的一致性比严格遵循某一条规则更重要。<br>参考资料<br>[1] [Typescript中interface与type的区别? - 可视化xld](http://www.bilibili.com/video/BV1KyrYY7EJc)<br>[2] [TypeScript中的Interface与Type:一字之差,妙用各异 - 百度智能云](https://cloud.baidu.com/article/3350043)<br>[3] [TypeScript 基础学习笔记:interface 与 type 的异同 - 腾讯云](https://cloud.tencent.com/developer/article/2427846)<br>[4] [Typescript中type和interface的区别是什么? - 百度教育](https://easylearn.baidu.com/edu-page/tiangong/questiondetail?id=1831323102614193529&fr=search)<br>[5] [TypeScript 中 Type 与 Interface 到底该怎么选?吃透这几点再也不纠结 - 葡萄城](http://zhuanlan.zhihu.com/p/1944364077804659360)<br>[6] [TypeScript中的interface和type:区别与使用场景 - 百度智能云](https://cloud.baidu.com/article/2835287)<br>[7] [TypeScript中interface与type的核心差异与实战选择 - CSDN博客](https://blog.csdn.net/weixin_29574585/article/details/163577516)<br>[8] [​​TypeScript 中 type 与 interface 的区别与最佳实践​​ - cary](http://zhuanlan.zhihu.com/p/1917952344680826458)<br>[9] [TypeScript开发之——实战37 interface 与 type - 荣行前端](http://www.bilibili.com/video/BV1wypCeGEt5)<br>[10] [在typescript中interface和type的区别和相同点 - 小不点灬 - 博客园 - 博客园](https://www.cnblogs.com/ximenchuifa/p/14896970.html)<br>[11] [TypeScript interface vs type 对比 - CSDN博客](https://blog.csdn.net/Bruce__taotao/article/details/147846230)<br>[12] [前端- TS中type和interface的区别 - 个人文章 - 51CTO博客](https://blog.51cto.com/u_16213711/14507489)<br>[13] [TypeScript中type与interface的异同 - 柒号玩家](http://quanmin.baidu.com/sv?source=share-h5&pd=qm_share_search&vid=9967827777466443369)<br>[14] [【Typescript 5极速进阶与实战指南】type 与 interface详细区别 - 前端进阶学习站](http://www.bilibili.com/video/BV1G2puzpEjs)<br>[15] [ts 和编译原理没了解过?那你可能还停留在初中级前端水平!一个视频掌握前端编译原理。| 高级前端面试 | 前端进阶 | 妙码前端进阶课 - 重生之我再干前端](http://www.bilibili.com/video/BV1gRVgzYEko?p=6)<br>[16] [typescript中通过type和interface定义类型的区别? - 海阔天空5210](http://www.bilibili.com/video/BV14N2oYfEps)<br>[17] [【前端必修技能】最新TypeScript 课程:秒懂 interface 和 type 的区别,轻松上手(前端开发/前端教程/前端面试/高薪就业) - Vueʵս֮·](http://www.bilibili.com/video/BV1QoVvzMEMQ?p=7)<br>[18] [【TypeScript】初识,基础类型,以及interface和type的使用 - CSDN博客](https://blog.csdn.net/2303_80072254/article/details/162041656)<br>[19] [别再死记硬背了!用几个真实场景帮你彻底搞懂TypeScript的interface和type - CSDN博客](https://blog.csdn.net/weixin_30522983/article/details/161908128)<br>[20] [typescript里interface和type都有哪些区别呢? - 千锋教育](https://www.zhihu.com/question/396825349/answer/3138882363)<br>[21] [解读Typescript中interface和type的用法及区别 - 脚本之家](https://www.jb51.net/javascript/335082rqt.htm)<br>[22] [TypeScript中interface 与 type的区别,你真的懂吗? - CSDN博客](https://blog.csdn.net/web220507/article/details/127105318)<br>[23] [TypeScript 中 Interface 与 Type 的深度对比分析 - CSDN博客](https://blog.csdn.net/m0_64344997/article/details/151756211)<br>[24] [TypeScript细碎知识点:interface和type的区别 - 梁飞宇 - 博客园 - 博客园](https://www.cnblogs.com/lxlx1798/articles/18126133)<br>[25] [TS中type和interface在类型声明时的区别 - 腾讯云](https://cloud.tencent.com/developer/article/2357671)<br>[26] [typescript中type和interface的区别有哪些 - 脚本之家](https://www.jb51.net/article/236714.htm)<br>[27] [TypeScript 开发抉择:interface 与 type 该如何选?深度解析与实践指南 - CSDN博客](https://blog.csdn.net/weixin_54447959/article/details/153009188)<br>[28] [Typescript 中的 type 和 interface 有什么区别??? - CSDN博客](https://blog.csdn.net/2401_86113453/article/details/142454340)<br>[29] [【TypeScript】TS中type比interface更香的场景(前端开发/项目实战/高薪就业/毕业设计/前端面试/AI/接单) - B站最美前端](http://www.bilibili.com/video/BV1eioWBNEQw?p=10)<br>[30] [【TypeScript】TS中type比interface更香的场景(前端开发/项目实战/高薪就业/毕业设计/前端面试/AI/接单) - B站最美前端](http://www.bilibili.com/video/BV1eioWBNEQw?p=9)<br>[31] [深入理解TypeScript:掌握interface和type的使用差异与应用场景(JS/ES6/vue3/前端教程/前端项目) - Vueʵս֮·](http://www.bilibili.com/video/BV1bP6JYVEge?p=8)<br><br>百度AI生成，内容仅供参考