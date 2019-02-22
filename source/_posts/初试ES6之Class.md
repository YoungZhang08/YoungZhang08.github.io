---
title: 初试ES6之Class
date: 2018-05-09 23:52:36
tags: 
- Class
categories: ES6
about:
---

## Class基础知识

### 语法：

```
// 方法一 Class声明
class Point {
    construstor(x, y) {
        this.x = x;
        this.y = y;
    }
    
    toString() {
        return '(' + this.x + ',' + this.y + ')';
    }
}

// 方法二 Class表达式
let person = new class {
    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }
}('张三');

person.sayName(); // "张三"
// person是一个立即执行的类的实例
```

### 类的数据类型是函数，因为类本身指向构造函数，类的方法都定义在prototype属性上面，可以使用Object.assign方法一次向类添加多个方法
<!-- more -->
### ！注意：
#### 1.在类的实例上调用方法，就是调用原型上的方法
#### 2.prototype对象的constructor属性直接指向类本身
#### 3.类的内部所有定义的方法都是不可枚举的
#### 4.类默认有constructor方法，如果没有显示定义，空的constructor方法会被默认添加
#### 5.constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象
#### 6.Class的name属性总是返回类名
```
class Point {
    constructor() {
        return Object.create(null);
    }
}
new Point instanceof Point   // false
```
#### 6.类必须使用new调用

---


## 敲黑板！！！划重点！！！

- #### 实例的属性除非显示定义在this对象上，否则都是定义在原型（class）上，即实例的自身属性都定义在this变量上，实例共享的属性和方法都定义在原型对象上也就是类上。
```
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
  
}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```
- #### 类的所有实例共享一个原型对象，也就是说可以通过实例的__proto__属性为class添加方法。不过，因为通过实例的__proto__属性改写原型会改变class的原始定义，会影响到所有实例，需注意！

```
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

--- 

__proto__不是语言本身特性，而是各大厂商实现的私有属性，不建议在生产环境中使用该属性，建议使用Object.getPrototypeOf方法来获取实例对象的原型，然后再添加源性方法/属性。

---

- #### 类不存在变量提升，必须保证子类在父类之后定义
- #### 私有方法的实现：
    ##### 1.方法名前加下划线（不保险），类的外部还是可以调到这个方法
    ##### 2.将私有方法移出模块（因为模块内部的方法都是对外可见的）
    ##### 3.利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值
- #### 私有属性的提案：
    ##### 1.使用#在属性名之前，使用时必须带有#一起使用
    ##### 2.私有属性可以指定初始值，在构造函数时进行初始化
    * #设置私有属性，也可以设置私有方法
    * 私有属性也可以设置getter和setter方法
- #### this指向：类的方法内部如果使用this，则默认指向类的实例，但是单独使用这个方法时，this会指向该方法运行时所在的环境，此时会因为找不到绑定对象而报错
    * 解决方法：
        1. 构造方法中指定this（最推荐！）
        2. 使用箭头函数
        3. 使用Proxy（未使用，详见ES6文档）
- #### Class的取值函数（getter）和存值函数（setter）是设置在属性的Descriptor对象上的
```
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```
- #### Class内部的方法前加上*，即为一个Generator函数（emmmmm，是时候又回去看看generator啦~）
- #### Class的静态方法：
    * 语法：static methodName() {}
    * 可以在类上调用，但是不能在实例上调用。如果静态方法中指定了this，则指向类。
    * 静态方法可以与非静态方法重名。
    * 父类的静态方法可以被子类继承，也可以从super对象上调用的。
- #### Class的静态属性和实例属性：
    * 静态属性指的是类本身的属性，而不是this上的属性
    * 类的实例属性可以用等式写在类的定义中
    ```
    class MyClass {
        myProp = 42;

        constructor() {
            console.log(this.myProp); // 42
        }
    }
    ```
    
    * 类的静态属性是在实例属性写法前面加上static关键字
    
    ```
    // 老写法
    class Foo {
        // ...
    }
    Foo.prop = 1;

    // 新写法
    class Foo {
        static prop = 1;
    }
    ```
- #### new.target属性用在构造函数之中可以确定构造函数是怎么调用的，正常返回new命令作用于的构造函数，如果构造函数不是通过new命令调用的，new.target返回undefined
    * Class内部调用new.target，返回当前Class
    * 子类继承父类时，new.target返回子类[可以用来写不能单独使用必须继承后才可使用的类]
    * 函数外部，使用new.target会报错


