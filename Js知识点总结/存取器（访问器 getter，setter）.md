
### 一. 来源

### 1.首先要了解==属性描述对象（共有6个）==

```JavaScript
{
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
  get: undefined,
  set: undefined
}
```
其中：
- get
`get`是一个函数，表示该属性的取值函数（getter），默认为`undefined`。

- set

`set`是一个函数，表示该属性的存值函数（setter），默认为`undefined`。

两者分别是操作js对象属性的值的 **取值** 和 **赋值**
[属性描述对象](https://wangdoc.com/javascript/stdlib/attributes)

### 二. 使用方式

#### 方式一 ： 作为构造器（class）中的属性值时

```JavaScript
class Example { 
	get hello() { 
		return "world"; 
	} 
} 
	
const obj = new Example(); 
console.log(obj.hello); 
// "world" 
console.log(Object.getOwnPropertyDescriptor(obj, "hello")); 
// undefined 
console.log( Object.getOwnPropertyDescriptor(Object.getPrototypeOf(obj), "hello"), );    // Object.getPrototypeOf(obj) 就是 obj.__proto__ 
// { configurable: true, enumerable: false, get: function get hello() { return 'world'; }, set: undefined }

```

#### 方式二：在`Object.defineProperty(obj,key,handler)` 中的handler中使用

``` JavaScript
var o = { a: 0 };

Object.defineProperty(o, "b", {
  get: function () {
    return this.a + 1;
  },
});

console.log(o.b); // Runs the getter, which yields a + 1 (which is 1)
```

``` JavaScript
var obj = Object.defineProperty({}, 'p', {
  get: function () {
    return 'getter';
  },
  set: function (value) {
    console.log('setter: ' + value);
  }
});

obj.p // "getter"
obj.p = 123 // "setter: 123"
```

#### 方式三：直接在`对象`（`{ }`）中定义属性

```JavaScript
var obj = {
  get p() {              //注意这里的写法
    return 'getter';
  },
  set p(value) {
    console.log('setter: ' + value);
  }
};
```

 [使用计算出的属性名](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get#%E4%BD%BF%E7%94%A8%E8%AE%A1%E7%AE%97%E5%87%BA%E7%9A%84%E5%B1%9E%E6%80%A7%E5%90%8D)

```javascript
var expr = "foo";

var obj = {
  get [expr]() {    // 动态参数
    return "bar";
  },
};

console.log(obj.foo); // "bar"
```
#### 方式四：在代理（`Proxy`）对象中==类似==的使用

- ==注==：*这种用法，不属于==存取器==，只是代码写法上有相似之处。故放在一起，作为比较，方便区分和记忆*

```javascript
var proxy = new Proxy({}, {
  get: function(target, propKey) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

or

```javascript
var proxy = new Proxy({}, {
  get(target, propKey) {        // ES6写法
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```


 **这里的get、set 是 拦截某个属性的读取 和 赋值操作 ，除此之外还有其他十一种；共计十三种：**

- **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。
- **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

	[Proxy详情文档](https://es6.ruanyifeng.com/#docs/proxy)



**总结**
- 1、注意各种场景下的编写方法，略有不同
- 2、前三种，属于常见的存取器的日常使用方式。最后一种是拦截对象原有的操作方法
- 3、前者其实都是实现了在给一个对象添加一个具名属性，但是细节上略有==不同==
     - 1: 当方式一使用 `get` 关键字时，属性将被定义在==**实例的原型**==上，当方式二使用[`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)时，属性将被定义在*==*实例自身**==上。
     - 2: 第二种写法，属性`p`的`configurable`和`enumerable`都为`false`，从而导致属性`p`是不可遍历的；第三种写法，属性`p`的`configurable`和`enumerable`都为`true`，因此属性`p`是可遍历的。实际开发中，写法三更常用。
- 4、一和三 的定义写法相同，二和四的定义写法相同

|     | 类型                                                  | 写法                                                                               | 属性将被定义的位置                                                                | 新增属性的特性                                           |
| --- | :-------------------------------------------------- | :------------------------------------------------------------------------------- | :----------------------------------------------------------------------- | ------------------------------------------------- |
| 方式一 | class 构造器<br>class Obj {<br>...<br>}                | 类函数写法  ==get propName( ){   return value;   }==                                  | new出来的**实例的原型**上，即：obj.\__proto\__ .propName  === Obj.prototype.propname | 由于是由构造器继承而来的，==不可枚举==（eg：  Object.keys() 无法获得该属性） |
| 方式二 | Object.definedPrototype(obj,'==propName==',handler) | 对象键值对写法  ==get:function( ){ return value; }==<br>*or*<br>get(){ return value;  } | 属性将被定义在**实例自身**上，即：obj.propname                                          | 不可写、==不可枚举==和不可配置（eg：  Object.keys() 无法获得该属性）     |
| 方式三 | 字面量声明对象 { }                                         | 类函数写法  ==get propName( ){   return value;   }==                                  |                                                                          | ==可枚举==，可配置<br>平时更加常用                             |
| 方式四 | { }                                                 | get:function( ){ <br>return value; }<br>==*or*==<br>get( ){ <br>return value; }  |                                                                          |                                                   |


  可见文档 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get)  [7. 存取器](https://wangdoc.com/javascript/stdlib/attributes)