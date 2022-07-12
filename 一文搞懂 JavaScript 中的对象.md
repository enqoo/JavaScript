JavaScript 中严格来说没有类的概念，只有对象（实例/实例对象），所有继承通过原型实现。

## 创建对象

### 对象字面量

对象字面量创建对象也叫单例模式。

```javascript
let Enqoo = {
  year: 2011,
  // 属性名可以是字符串或数值。数值属性会自动转换为字符串
  10: true,
  "server": "design"
}
```

在使用对象字面量表示法定义对象时，并不会实际调用 Object 构造函数。

### Object 构造函数

使用 Object 类型创建对象。

```javascript
let Enqoo = new Object();
// 赋值
Enqoo.age = 10;
```

### 工厂模式

```javascript
// 工厂模式创建对象, 用函数封装特定接口
function EnqooFactory(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function () {
        console.log(this.name);
    };
    return o;
}
var Enqoo1 = EnqooFactory("小红", 20, "设计师");
Enqoo1.sayName(); // 小红

// 工厂模式的问题: 无法使用 constructor 或 instanceof 识别函数的对象类型
console.log(Enqoo1.constructor == factoryFun); //false
console.log(Enqoo1 instanceof factoryFun); //false
console.log(Enqoo1 instanceof Object); //true
```

### 构造函数模式

像 Object、Array 这样的原生对象都是用构造函数创建的，运行时自动出现在执行环境中。还可以自定义构造函数，定义对象类型的属性和方法。

```javascript
function Enqoo(name) {
    this.name = name;   // this.Name 改为var name = name 也可以，属性就是类（函数）作用域的变量。属性可以等任何值，赋予函数就是方法。
  
    // 构造方法，this 把私有变量设为特权方法.
    // 当一个函数用作构造函数时（使用new关键字），它的 this 被绑定到正在构造的新对象。
    this.sayName = function(){
        console.log(this.name);
    }
}
// 任何函数,只要通过new操作符来调用,那它就可以作为构造函数
person1 = new Enqoo("当作构造函数使用");
person1.sayName();

person2 = new Enqoo("当作构造函数使用");
person2.sayName();

//作为普通函数调用
Enqoo("普通函数调用");
window.sayName();// 当在全局作用域调用一个函数时, this对象总是指向Global对象在浏览器中为windows

// 在另一个对象的作用域中调用
var o = new Object();
Enqoo.call(o, "调用另一个对象");
o.sayName();

// 对象都有一个constructor 属性, 用来识别对象类型。或是 instanceof 更方便些
console.log(person1.constructor == Enqoo); //true
console.log(person1 instanceof Enqoo); // true
console.log(person1 instanceof Object);//true


// 函数是对象,因此每定义一个函数,也就是实例化了一个对象, 上述构造也可以这样定义:
function Enqoo(name) {
   this.name = name;
   this.sayName = new Function("console.log(this.name);");
}

//构造函数的缺点: 每个方法都要在每个实例上重新创建一遍, 两个方法不是同一个Function实例
console.log(person1.sayName == person2.sayName); //false
// 可以把函数定义转移函数定义移到外面去
```

### 原型模式

函数是对象，每次 function 一个函数，其实是 new Function 类型的实例对象。所以每个函数都会创建一个 prototype 属性，属性是一个指针，指向一个**原型对象**（默认情况下只有 `constructor`  属性的对象，属性 `constructor` 指向函数自身）。

而实例对象（ new F() ）内部隐藏的内部指针 [[Prototype]] 指向构造函数的原型对象 prototype ，即构造函数的 `prototype` 原型对象等于该实例对象的`__proto__`对象。构造函数的对象（引用类型）通过不断链接原型对象中`__proto__`的，继而继承自 Object，从而构成原型链，最终到达 null。

当使用 new F() 创建一个对象时，函数内部创建一个  `this`对象，并返回  `return this`。

```javascript
var Enqoo = function (name) {
    // 创建一个空对象，属性和方法增加到此 this 上。new 后即可访问
    // var this = {};

    // add properties and methods
    this.name = name;
    this.say = function () {
        return "I am " + this.name;
    };

    //return this;
};

let brand = new Enqoo(); // 访问函数内部 this 的属性和方法
```

函数（函数即是对象）有个默认 `prototype` 属性，指向一个对象，在对象上增加属性和方法。

```javascript
// 方法直接增加到函数中，new F() 会创建多份。因此最好直接增加到原型上。（摘录来自: wizardforcel. “JavaScript 模式。” ）
Enqoo.prototype.say = function () {
    return "I am " + this.name;
};
```

当 new F() 时，内部 `this` 指向 `F. prototype`，所以`new`出来的对象可以访问原型对象上的方法和属性。

```javascript
function Enqoo(){}

console.log( Enqoo.prototype ) // { constructor:f Enqoo(), __proto__: Object }
console.log( Enqoo.prototype.constructor === Enqoo ); // true
/*										
|
|new 构造函数的实例对象有个隐藏的内部指针 [[Prototype]] (enqoo.[[Prototype]]) 指向构造函数的原型对象 Enqoo.prototype。通过 __proto__ 访问。
| [[Prototype]] 属性要么等于对象，要么是 null。即继承对象到达 null。利用此原理实现继承
|
*/
let enqoo = new Enqoo();

// 与构造函数的 prototype 相等
console.log(enqoo.__proto__ ); // constructor: f Enqoo(), __proto__: Object

// 继续查找上一级构造函数构成的对象
enqoo.__proto__.__proto__ === Object.prototype === Enqoo.prototype.__proto__ 

//  Object.getPrototypeOf()，返回参数的内部特性 [[Prototype]]的值
enqoo.__proto__ === Enqoo.prototype  === Object.getPrototypeOf(enqoo) // true

// 最终达到 null
enqoo.__proto__.__proto__.__proto__ //  = Enqoo.prototype.__proto__.__proto__
```

函数即是对象，可直接增加属性和方法

```javascript
Enqoo.year = 2011;
Enqoo.getYear = function(){}
```

所有原生引用类型（以大写开头如 Object、Array、String等）都是在其构造函数的原型上定义了方法。

```javascript
console.log(typeof Document.prototype.getElementById); // function
console.log(typeof Array.prototype.sort); // function
```

### 使用 `constructor` 属性

默认不改变`prototype`原型对象的情况下，可使用 `constructor` 属性来创建一个新对象，该对象使用与现有对象相同的构造器。

```javascript
function Enqoo() {} // 默认: Enqoo.prototype = { constructor: Enqoo }
let company = new Enqoo(); // 继承自 {constructor: Enqoo}

// 使用现有对象的 constructor 创建新对象
let company2 = new company.constructor();
```

https://zh.javascript.info/function-prototype

### Class 类

在 JavaScript 中，类是一种构造函数，类中的方法亦是在原型对象中实现。实例后不能直接遍历/枚举其中的方法：https://zh.javascript.info/class#bu-jin-jin-shi-yu-fa-tang

可用使用类`prototype`遍历：

https://stackoverflow.com/questions/37771418/iterate-through-methods-and-properties-of-an-es6-class

```javascript
class Enqoo { // = function Enqoo
  prop = value; // 属性

  constructor(...) { // 构造器 = Enqoo.prototype.constructor()
    // ...
  }

  method(...) {} // method

  get something(...) {} // getter 方法
  set something(...) {} // setter 方法

  [Symbol.iterator]() {} // 有计算名称（computed name）的方法（此处为 symbol）
  // ...
}
```



## 原型继承

Javascript 中所有原生方法都是通过原型对象实现的。

###  通过旧式`__proto__`继承

通过子对象的 `__proto__` 继承父对象。

```javascript
let Enqoo = {
  design: true
}
let Qooui = {
  develope: true
}

Qooui.__proto__ = Enqoo; // 设置 Enqoo.[[Prototype]] = Qooui
Qooui.design; // true 继承自 Enqoo 对象
```

或

```javascript
let Enqoo = {
  design: 'ul',
  service(){
    console.log(this.design);
  }
}
let Qooui = {
  develope: true,
  __proto__: Enqoo
}

Qooui.service(); // ul
```

### 通过 prototype 原型链继承

子构造函数的 `prototype` 属性继承父对象（ new + 构造函数）。

```javascript
let Enqoo = {
  design: true
}
function Qooui(){}

Qooui.prototype = Enqoo; // 如果 Enqoo为构造函数，可用 new Enqoo() 构建对象 
  
let brand = new Qooui(); // brand.__proto__ == Enqoo
brand.design; // true
```

### Object.create()

使用现有的对象来创建新对象的`__proto__`。

```javascript
let Enqoo = {
  design: 'yes'
}
// 第二个参数属性描述器，可以为新对象提供额外的属性
let company = Object.create(Enqoo, { service:{value:true} });
company.design; // "yes"
company.service; // true
```

### 盗用构造函数

也称为“对象伪装”或“经典继承”。在子类构造函数中调用父类构造函数。

```javascript
function Enqoo(name){
  this.service = "design"
  this.name = name;
}
function subBrand(){
  // 继承 Enqoo
  Enqoo.call(this, "Qooui")//传递参数
}
let company = new subBrand();
company.service; // design

let company2 = new subBrand();
company2.name; // Qooui
```

盗用构造函数缺点，必须在构造函数中定义方法，函数不能重用。

### 组合继承

也叫伪经典继承。原型链继承共性的属性和方法，盗用构造函数继承实例属性。

```javascript
function Enqoo(name) {
    this.service = "design"
    this.name = name;
}
Enqoo.prototype.sayName = function () {
    alert(this.name);
}

function subBrand(name, age) {
    // 盗用构造函数继承实例属性
    Enqoo.call(this, name)

    // 子类属性
    this.age = age;
}


// 原型链继承原型上的共享的属性和方法
subBrand.prototype = new Enqoo();

// 子类方法要放在继承父类后
subBrand.prototype.sayAge = function () {
    alert(this.age);
}

let company = new subBrand("Qooui", 12);
company.name; // Qooui 继承盗用构造函数的属性
company.sayName() // 继承原型链上的方法

let company2 = new subBrand("Qomla", 14);
company2.sayAge();
```

### 原型式继承

创建一个临时构造函数，将传入的对象赋值给这个构造函数的原型，然后返回这个临时类型的一个实例。

```javascript
function Enqoo(o) {
    function F(){}
    F.prototype = o;
    return new F;
}

let company = {
    name: "Enqoo",
    brands: ["Qooui", "Qomla"]
}

let brand = Enqoo(company);

brand.name; // Enqoo

brand.brands.push("Qokit");

company.brands; //  ["Qooui", "Qomla", "Qokit"]
```

### 寄生式继承

类似于寄生构造函数和工厂模式:创建一个实现继承的函数，以某种方式增强对象，然后返回这个对象。

```javascript
function Enqoo(o) {
    function F(){}
    F.prototype = o;
    return new F;
}

function createAnother(original){
    let clone = Enqoo(original);

    clone.sayHi = function(){
        console.log('hi');
    }
    return clone;
}

let company = {
    name: "Enqoo",
    brands: ["Qooui", "Qomla"]
}

let brand = createAnother(company);

console.log(brand); // 包含新增的对象方法和继承属性和方法
```

### 寄生式组合继承

解决组合两次调用父类构造函数问题。

```javascript
function object(o) {
    function F(){}
    F.prototype = o;
    return new F;
}

function superEnqoo(name) {
    this.service = "design"
    this.name = name;
}
superEnqoo.prototype.sayName = function () {
    alert(this.name);
}

function subBrand(name, age) {
    // 盗用构造函数继承实例属性
    Enqoo.call(this, name)

    // 子类属性
    this.age = age;
}

/**
 * 
 * @param subType 子类构造函数
 * @param superType 父类构造函数
 */
function inheritPrototype(subType, superType) {
    let prototype = object(superType.prototype); // 创建对象 
    prototype.constructor = subType; // 增强对象
    subType.prototype = prototype; // 赋值对象
}

inheritPrototype(subBrand, superEnqoo);


subBrand.prototype.sayAge = function () {
    alert(this.age);
}
```



## 模块



**延伸阅读：**

《原型，继承》：https://zh.javascript.info/prototypes

《对象原型》：https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes

《对象模型的细节》：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Details_of_the_Object_Model

《继承与原型链》:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

