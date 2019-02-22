---
title: JS深拷贝浅拷贝
date: 2018-05-17 13:51:43
tags: 
- 对象拷贝
categories: ES5 ES6
about:
---
### 深拷贝和浅拷贝主要区别是在内存中存储类型（堆栈）不同。

> 但其实js中没有严格意义上区分堆栈内存，因此可以粗暴的认为所有的数据类型都存储在堆内存，但是有些场景下还是需要利用栈数据结构的思维来实现一些思维，比如，js的执行上下文就采用了栈结构实现环境栈（又名函数调用栈）。

## 堆栈的区别
> 栈，先进后出(FILO)的存储方式，按值存放可直接访问。堆，树状结构，按引用访问。
<!--more-->
## js中的数据类型
1. 基本数据类型：String，Number，Null，Undefined，Boolean。五中基本数据类型的变量直接按值存放，存在栈内存中，可直接访问。
2. 引用类型：Object，Array，Function。值大小不固定，栈内存中的变量值存的是地址，指向堆内存中的对象。访问引用类型的值时，首先从栈中获得该对象的地址指针，然后再从堆内存中取得数据。
* 见下图理解：

![](https://note.youdao.com/yws/public/resource/9274007941c429664226c48c2ba441e5/xmlnote/WEBRESOURCEe9e63c24c049cb8c31d381f59aafc906/2976)

### 说拷贝之前还想再说一下js中的按值传递和按引用传递
#### 按值传递
> 高程书中写道：ECMAScript中所有函数的参数都是按值传递的。也就是说，把函数外部的值复制给函数内部的参数，就和把值从一个变量复制到另一个变量一样。可以把ECMAScript函数的参数想象成局部变量。

举个例子：

```
function foo(v) {
    v = 2;
    return v;
}

var value = 1;
foo(value); // 2
console.log(value) // 1
```
可以看到，value被传递到foo函数中的时候，相当于把实参变量value的值拷贝给形参变量v，函数内部修改的都是形参变量v的值，不会影响原来的变量value的值。

#### 按引用传递
> 像上述的基本数据类型的值的拷贝还是很好理解的，但是当值的数据类型变得复杂，上面的拷贝方式就会产生性能上的影响，所以出现了另外一种传递方式——按引用传递。所谓引用传递，就是传递对象的引用，函数内部对参数的修改会反映在函数外部，因为两者引用的是同一个对象。

举个例子：

```
function change(person) {
    person.age = 25;
    
    return person;
}

var obj1 = {
    name: 'young',
    age: 21
};
var obj2 = change(obj1);
console.log(obj1); // {name: 'young', age: 25}
console.log(obj2); // {name: 'young', age: 25}
```
等一下，和高程书说的好像有点不一样，这样子的话看着像是按引用传递了······稍安勿躁~

#### 共享传递
> 共享传递是指，在传递对象的时候，传递对象的引用的副本。注意！！！按引用传递是指传递对象的引用，按共享传递是指传递对象的引用的副本。

再看个例子：

```
function change(person) {
    person.age = 25;
    person = {
        name: 'miaorenjie',
        age: 20
    };
    
    return person;
}

var obj1 = {
    name: 'young',
    age: 21
};
var obj2 = change(obj1);
console.log(obj1); // {name: 'young', age: 25}
console.log(obj2); // {name: 'miaorenjie', age: 20}
```
所以修改person.age是通过引用找到原值，并且影响原值的，并不是通过person，后面直接修改person，是更改了person的指向，也就是重新拷贝了指向内存的地址，原本指向obj1的指针就变为了指向新的对象，所以，可以看到的是obj1的属性改变是通过引用修改的，而如果说是按引用传递的话，person的修改也是会反映在obj1上的。所以，总结一下，我们可以这么理解：
> 参数如果是基本数据类型，就是按值传递，如果是引用类型，就是按共享传递。因为不论拷贝值副本还是地址副本始终也是拷贝的值，所以红宝书告诉你说参数都是按值传递~

### 浅拷贝&深拷贝
首先，深拷贝和浅拷贝在js中是对引用类型而言的。

> 浅拷贝：指新拷贝的对象和原对象指向同一个地址，改变其中一个变量的内容都会反映在另外一个变量身上，也就是拷贝的原对象的内存地址。
深拷贝：指重新分配内存给新拷贝的对象，将原对象的各个属性复制进去，对拷贝对象和原对象各自的操作互不影响。

接下来看具体实现吧，终于切入正题了~
1. 赋值：

```
var obj = {
    name : 'miaorenjie',
    age: 20,
    like: ['black', 'white']
};

var obj1 = obj;
obj1.name = 'young';
obj1.like[0] = 'pink';

console.log(obj); // { name: 'young', age: 20, like: [ 'pink', 'blue' ] }
console.log(obj1); // { name: 'young', age: 20, like: [ 'pink', 'blue' ] }
```
可以看到的是，赋值操作是纯粹的复制原对象的引用，所以在修改新对象的原始数据类型和引用类型的值之后的变化也都反映在了原对象上。

2. 浅拷贝（简单的复制引用）

```
function shallowCopy(src) {
    var result = {};
    for(var i in src) {
        if(src.hasOwnProperty(i)) {
            result[i] = src[i];
        }
    }
    
    return result;
}

var obj = {
    name : 'miaorenjie',
    age: 20,
    like: ['black', 'white']
}

var obj2 = shallowCopy(obj);
obj2.age = 21;
obj2.like[1] = 'blue';
console.log(obj); // { name: 'miaorenjie', age: 20, like: [ 'black', 'blue' ] }
console.log(obj2); // { name: 'miaorenjie', age: 21, like: [ 'black', 'blue' ] }
```
可以看到，新复制的对象好像并没有和原对象共用内存呀，要不然怎么基本数据类型没有随之修改。如果你也产生了这个疑问，那么恭喜，你和小编走入了同一个误区。此处所实现的浅复制，指的是循环复制原对象的属性时，进行的一层浅复制，也就是在循环赋值到某个具体的属性的时候，对这个具体的属性进行复制，如果是原始数据类型就复制的属性值，如果是引用类型就复制引用类型的属性值——对象的引用，所以基本数据类型是没有共用内存的，只有原对象中的对象才是共用内存的，所以只有引用类型的值在互相影响。

2. Object.assign()
> Object.assign(target, ...sources)方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。[target:目标对象，sources:源对象] 

- 浅拷贝
> 因为 Object.assign()拷贝的是属性值。。假如源对象的属性值是一个指向对象的引用，它就拷贝那个引用值，也就是源对象中的引用属性会和目标对象的该属性共用内存，两者的后续修改都会影响到对方，这时候获得的就是源对象的浅复制。

```
// 第一种假如
let obj1 = { a: 0 , b: { c: 0}};
let obj2 = Object.assign({}, obj1);
obj1.a = 1;
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 0}}
console.log(JSON.stringify(obj2)); // { a: 0, b: { c: 0}}
obj2.a = 2;
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 0}}
console.log(JSON.stringify(obj2)); // { a: 2, b: { c: 0}}

// 第二种假如
obj2.b.c = 3;
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 3}}
console.log(JSON.stringify(obj2)); // { a: 2, b: { c: 3}}
```

- 拷贝Symbol类型的属性

```
var o1 = { a: 1 };
var o2 = { [Symbol('foo')]: 2 };

var obj = Object.assign({}, o1, o2);
console.log(obj); // { a : 1, [Symbol("foo")]: 2 } (cf. bug 1207182 on Firefox)
Object.getOwnPropertySymbols(obj); // [Symbol(foo)]
```
- 原始类型会被包装为对象
> Object.assign()会忽略null或undefined的源值，因为原始类型会被包装成对象，但是null和undefined无法转成对象，只有字符串的包装对象才可能有自身可枚举属性。包装后的对象属性值对应的键是它在源对象中的索引。

```
var v1 = "abc";
var v2 = true;
var v3 = 10;
var v4 = Symbol("foo")

var obj = Object.assign({}, v1, null, v2, undefined, v3, v4); 
// 原始类型会被包装，null 和 undefined 会被忽略。
// 注意，只有字符串的包装对象才可能有自身可枚举属性。
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```
-  继承属性和不可枚举属性不可拷贝
> Object.assign 方法只会拷贝源对象自身的并且可枚举的属性到目标对象。该方法使用源对象的[[Get]]和目标对象的[[Set]]，所以它会调用相关 getter 和 setter。因此，它是分配属性，而不仅仅是复制或定义新的属性。如果合并源包含getter，这可能使其不适合将新属性合并到原型中。为了将属性定义（包括其可枚举性）复制到原型，应使用Object.getOwnPropertyDescriptor()和Object.defineProperty() 。

```
var obj = Object.create({foo: 1}, { // foo 是个继承属性。
    bar: {
        value: 2  // bar 是个不可枚举属性。
    },
    baz: {
        value: 3,
        enumerable: true  // baz 是个自身可枚举属性。
    }
});

var copy = Object.assign({}, obj);
console.log(copy); // { baz: 3 }
```
- 属性覆盖（可用来合并对象）
> 如果目标对象中的属性具有相同的键，则属性将被源中的属性覆盖。后来的源的属性将类似地覆盖早先的属性。

```
var o1 = { a: 1, b: 1, c: 1 };
var o2 = { b: 2, c: 2 };
var o3 = { c: 3 };

var obj = Object.assign({}, o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
```
- 异常会打断后续拷贝任务
> 在出现错误的情况下，例如，如果属性不可写，会引发TypeError，如果在引发错误之前添加了任何属性，则可以更改target对象。

```
var target = Object.defineProperty({}, "foo", {
    value: 1,
    writable: false
}); // target 的 foo 属性是个只读属性。

Object.assign(target, {bar: 2}, {foo2: 3, foo: 3, foo3: 3}, {baz: 4});
// TypeError: "foo" is read-only
// 注意这个异常是在拷贝第二个源对象的第二个属性时发生的。

console.log(target.bar);  // 2，说明第一个源对象拷贝成功了。
console.log(target.foo2); // 3，说明第二个源对象的第一个属性也拷贝成功了。
console.log(target.foo);  // 1，只读属性不能被覆盖，所以第二个源对象的第二个属性拷贝失败了。
console.log(target.foo3); // undefined，异常之后 assign 方法就退出了，第三个属性是不会被拷贝到的。
console.log(target.baz);  // undefined，第三个源对象更是不会被拷贝到的。
```
3. 深拷贝
> 上面也说了，浅复制复制的是引用类型的地址，并没有开辟新的内存空间，而深复制是开辟新的内存来存放新对象，源对象和新对象对应的是两个不同的地址，修改其中一个对象的属性，不会改变另外一个对象的属性。深复制的几种实现：

- 递归解析
> 因为使用递归，性能会不如浅拷贝
```
function deepCopy(src, res) {
    var res = res || {};
    for(var i in src) {
        if(src.hasOwnProperty(i)) {
            if(typeof src[i] === 'object') {
                res[i] = (src[i].constructor === Array) ? [] : {};
                deepCopy(src[i], res[i]);
            } else {
                res[i] = src[i];
            }
        }
    }
    return res;
}

var obj1 = {};
deepCopy(obj, obj1);
obj1.age = 21;
obj1.like[1] = 'blue';

console.log(obj); // { name: 'miaorenjie', age: 20, like: [ 'black', 'white' ] }
console.log(obj1); // { name: 'miaorenjie', age: 21, like: [ 'black', 'blue' ] }
```
- JSON解析
> 局限性：如果被拷贝的对象中有function，则拷贝之后的对象就会丢失这个function，原型链就没了；如果被拷贝的对象中有正则表达式，则拷贝之后的对象正则表达式会变成Object

```
var obj = {
    name : 'miaorenjie',
    age: 20,
    like: ['black', 'white']
}

var result = JSON.parse(JSON.stringify(obj));
result.age = 21;
result.name = 'young';
result.like.push('pink');
console.log(obj); // { name: 'miaorenjie', age: 20, like: [ 'black', 'white' ] }
console.log(result); // { name: 'young', age: 21, like: [ 'black', 'white', 'pink' ] }
```
- Object.assign()（当对象只有一层的时候使用）
> 假如源对象的属性值不包含引用类型，那该方法返回的值就是一个与源对象相同但不在同一块内存的另一个对象，源对象和新对象的后续修改是互不影响的，也就是返回了源对象的深拷贝。

```
obj1 = { a: 0 , b: { c: 0}};
let obj3 = JSON.parse(JSON.stringify(obj1));
obj1.a = 4;
obj1.b.c = 4;
console.log(JSON.stringify(obj3)); // { a: 0, b: { c: 0}}
```