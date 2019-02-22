---
title: 从ES5&ES6两方看继承(一)
date: 2018-05-10 21:56:36
tags:
- 对象 
- 原型 
- 继承
categories: ES5
about:
---


> ##### 原型和继承，是学习JS过程中相当重要并且难懂的点，第一次过这块的时候，完全是十脸懵逼的。之后因为准备春招，把这块的整个体系重新拜读了不下三次，才得以勉强面试。最近，也在学习ES6相关，看到Class的extend继承，就想对ES5的继承和ES6的继承做一比对和整理。好啦~正文开始······


### 首先，ES5继承

整理一下构造函数、原型对象和实例之间的关系：构造函数中有一个prototype的属性指向它自己的原型(prototype)对象，而原型对象中又有一个constructor属性指回构造函数，原型对象中定义实例共享的属性和方法，实例对象也总有一个prototype属性指向自己的默认原型对象。实例对象和原型对象均有__prpto__属性，它们的__proto__也都指向它的构造函数的原型对象。
> 注：JS中所有对象都有自己的__proto__属性，隐式指针，也就是后文中的[[prototype]]

<!--more-->

#### 1. ES5继承之原型链[ 核心：将父类的实例作为子类的原型，即重写子类的原型 ]

```
// 实现模式
function Father() {
    this.property = true;
}

Father.prototype.getFatherPro = function() {
    return this.property;
}

function Son() {
    this.sonProperty = false;
}

// Son继承了Father
Son.prototype = new Father();

Son.prototype.getSonPro = function() {
    return this.sonProperty;
}

var instance = new Son();
console.log(instance.getFatherPro); // true

```
![](https://note.youdao.com/yws/public/resource/c1554b4563a4504a4b47be66421ba010/xmlnote/WEBRESOURCE1c5ea84c6c60b09c19ba3974de065153/2699)

> 这个代码中，我们改写了Son的原型，这个新原型是Father的实例，而Father的实例的prototype是指向Father Prototype的，所以新原型具有Father的实例拥有的所有属性（property）和方法，而我们后面也给它的新原型添加了方法getSonPro()，所以，在new出来Son的实例instance时，instance的prototype指向Son Prototype，并拥有Son的属性sonProperty和Son Prototype上的属性和方法，自然就可以调用getSonPro并打印出结果。

#### ！！！敲黑板啦~——默认原型，前面的原型链还少一环，就是Father Prototype的[[prototype]]指向Object Prototype，而Object Prototype的[[prototype]]指向null。

##### 确定原型和实例的关系
1. instanceOf
2. isPrototypeOf()

> 原型链继承的问题：
> 1. 因为包含引用类型值的原型属性会被所有实例共享，所以我们一般都是在构造函数中定义属性。通过原型来继承的时候，因为子类的原型实际上是父类的实例，所以父类实例的属性就会变成子类的原型属性，那么子类的所有实例都会共享子类原型上的属性也就是父类上的属性，当子类实例访问或者改变这个属性的时候，也会反映在其他实例上。
> 2. 在创建子类的实例时不能向父类的构造函数中传递参数。

---

#### 2. ES5继承之借用构造函数[ 核心：子类的构造函数中调用父类的构造函数 ]

```
function Father() {
    this.arr = [1, 2, 3, 4];
}

function Son() {
    // 继承Father
    Father.call(this);
}

var instance1 = new Son();
instance1.arr.push(5);
console.log(instance1.arr); // [1, 2, 3, 4, 5]

var instance2 = new Son();
console.log(instance2.arr); // [1, 2, 3, 4]
```
> call和apply方法的作用就是在（将来）新创建的实例对象上执行构造函数，所以每个实例对象都会有自己的属性副本，不会影响到所有实例都是相同的属性。而且，借用构造函数还有一个大优势就是可以在子类构造函数中向父类构造函数传参。


```
function Father(name) {
    this.name = name;
}

function Son() {
    // 继承Father，传入参数
    Father.call(this, 'zyy');
    
    // 实例属性
    this.age = 21;
}

var instance = new Son();
console.log(instance.name); // zyy
console.log(instance.age); // 21
```
> 构造函数的问题：
> 1. 方法都在构造函数中定义，函数复用就无法实现。
> 2. 在父类原型中定义的方法对于子类而言是不可见的，所以所有类型只能使用构造函数模式。

#### 3. ES5继承之组合继承[ 核心：结合构造函数和原型链，即使用原型链实现对原型属性和方法的继承，通过构造函数实现对实例属性的继承 ]
*以最典型的AJAX为例*

```
function Ajax(obj) {
    this.url = obj.url;
    this.async = obj.async;
    this.data = obj.data || '';
    this.method = obj.method || 'GET';
    this.dataType = obj.dataTpye;
    this.success = obj.success;
    this.error = obj.error;
    this.timeout = obj.timeout || 0;
    this.xml = null;
    this.init();
}

Ajax.prototype = {
    constructor: Ajax,
    init: function() {
        this.createXml();
    },
    createXml: function() {
        ······
        this.spliceParams();
        this.dataType();
    },
    spliceParams: function() {
        ······
        this.data = arr;
    },
    dataType: function() {
        ······
    }
}

window.Ajax = function(data) {
    return new Ajax(data);
}
```
> 组合继承的优点：酱紫结合的结果就是所有实例都共享原型对象上的方法，但是又拥有自己独有的属性~

#### 4. ES5继承之原型式继承[ 核心：借助原型基于已有的对象创建新对象 ]

```
function createObj(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

var person = {
    name: 'zyy',
    friends: ['miao', 'pm']
}

var person1 = createObj(person);
var person2 = createObj(person);

person1.name = 'person1';
console.log(person1.name); // person1
console.log(person2.name); // zyy

person1.friends.push('tuo');
console.log(person2.friends); // ["miao", "pm", "tuo"]
```
> 原型继承的问题：
> 1. 包含引用类型的属性值始终都会共享相应的值，这点跟原型链继承一样。
> 2. 上例代码中的person2的friends和person1的一样，而name不同就是因为，包含引用类型的属性值会共享。

#### 5. ES5继承之寄生式继承[ 仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象 ]

```
function createObj (o) {
    var clone = Object.create(o);
    clone.sayHi = function () {
        console.log('hi');
    }
    return clone;
}

var person = {
    name: 'zyy',
    friends: ['miao', 'pm', 'tuo']
};

var anotherPerson = createObj(person);
anotherPerson.sayHi(); // hi
```
> 寄生式继承的问题：跟借用构造函数模式一样，每次创建对象都会创建一遍方法

#### 6. ES5继承之寄生组合式继承
> 组合继承最大的缺点是会调用两次父构造函数。
一次是设置子类型实例的原型的时候，一次在创建子类型实例的时候。


```
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

// 当我们使用的时候：
prototype(Child, Parent);
```
> 这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。

