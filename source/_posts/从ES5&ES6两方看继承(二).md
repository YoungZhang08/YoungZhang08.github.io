---
title: 从ES5&ES6两方看继承(二)
date: 2018-05-18 15:13:58
tags: 
- 对象
- 继承
categories: ES6
about:
---

## 接着上文，看ES6继承之Class & extends

### 1.基本语法

```
class Point {
}

// ColorPoint 
class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y); // 调用父类的constructor(x, y)
        this.color = color;
    }

    toString() {
        return this.color + ' ' + super.toString(); // 调用父类的toString()
    }
}
```
super方法表示父类构造函数，调用时会新建父类的this对象。只有super方法才可以返回父类实例，所以调用super之后子类才可以使用this关键字。

<!--more-->

> 注意：子类必须在constructor方法中调用super方法，否则新建实例的时候会报错。这是因为ES6的继承机制是酱紫的：先创建父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。也就是说子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其加工，加上子类自己的实例属性和方法。如果不调用super方法子类就得不到this对象。最后，父类的静态方法也会被子类继承。

扯一扯ES5的继承机制：先创建子类的实例对象this，然后将父类的方法添加到子类的实例对象this上面（Parent.apply(this)）。

### 2. Object.getPrototypeOf()
> 用来从子类上获取父类，因此可以用来判断一个类是否继承了另一个类。

```
Object.getPrototypeOf(ColorPoint) === Point; // true
```

### 3. super关键字
- 作为函数调用时，代表父类的构造函数且只能在子类的构造函数中。即使super代表了父类的构造函数，但是返回的是子类的实例，也就是其内部的this指的是子类，所以super相当于是 ++**父类.prototype.constructor.call(this)**++
- 作为对象时，++在普通方法中，指向父类的原型对象++，所以定义在父类实例上的属性和方法无法通过super调用。规定在子类普通方法中通过super调用父类方法时，内部的this指向当前的子类实例。因为this指向子类实例，所以++如果在子类的constructor方法中通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性；在静态方法中，指向父类，而不是父类的原型对象++。

### 4.类的prototype和__proto__
> 作为一个对象，子类的原型（__proto__属性）是父类。

> 作为一个构造函数，子类的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

### 5.实例的__proto__属性
> 子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型。因此，通过子类实例的__proto__.__proto__属性，可以修改父类实例的行为。

### 6.原生构造函数的继承
> Boolean()、Number()、String()、Array()、Date()、Function()、RegExp()、Error()、Object()以上这些原生构造函数是无法继承的。
原生构造函数的this无法绑定，导致拿不到内部属性。因为ES5中是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数。

```
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});

var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```
之所以会发生这种情况，是因为子类无法获得原生构造函数的内部属性，通过Array.apply()或者分配给原型对象都不行。原生构造函数会忽略apply方法传入的this，也就是说，原生构造函数的this无法绑定，导致拿不到内部属性。

> ES6允许继承原生构造函数定义子类，因为ES6是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。

```
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined
```