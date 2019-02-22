---
title: 腾讯now直播一面笔试题再探
date: 2018-05-18 16:36:01
tags:
- 类
- 对象
categories: ES5 ES6
about:
---

emmmmm，现在回过头来看腾讯面试题了，哇~ 想想当时的寄己，真的是基础不过关，也就只能现在弥补了！
看题——

```
写一个构造方法Point，要求实现以下功能：
let point = new Point(100, 100);
let anotherPoint = point.move(-10, 10);
console.info(anotherPoint.coor); // {x: 90, y:110}
console.info(point.coor); // {x: 100, y: 100}
```
<!--more-->

> 首先说一下思路：Point类有一个成员方法move()和coor()，move()接收两个参数(a, b)，并且返回一个新的Point对象，新对象的值是(当前对象+a, 当前对象+b)，coor()返回当前对象的值。


```
// 方法一
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
        
    get coor() {
        return {
            x: this.x,
            y: this.y
        }
    }

    move(a, b) {
        return new Point(this.x + a, this.y + b);
    }
}
    
let point = new Point(100,100);
let anotherPoint = point.move(-10,10);
console.info(anotherPoint.coor); // { x: 90, y: 110 }
console.info(point.coor); // { x: 100, y: 100 }
```

```
// 方法二
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.coor = {
            x: this.x,
            y: this.y
        }
    }

    move(a, b) {
        return new Point(this.x + a, this.y + b);
    }
}

let point = new Point(100,100);
let anotherPoint = point.move(-10,10);
console.info(anotherPoint.coor); // { x: 90, y: 110 }
console.info(point.coor); // { x: 100, y: 100 }
```

