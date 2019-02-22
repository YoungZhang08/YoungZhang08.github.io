---
title: 由一道算法带进ES6之SetMap
date: 2018-05-08 17:00:42
categories: ES6 
tags: 
- Set 
- Map
---

# Set:
## 构造函数（可生成Set数据结构），类似数组，成员值唯一。

## Set实例具有的属性：
### 1.Set.prototype.constructor:默认Set
### 2.Set.prototype.size:成员总数

<!-- more -->

## Set实例的操作和遍历方法：
### 1.add(value)：添加，返回Set结构
### 2.delete(value)：删除，返回布尔值，表示是否删除成功
### 3.has(value)：返回布尔值，表示是否为Set成员
### 4.clear()：清除所有成员，无返回值
### 5.keys()：返回键名
### 6.values()：返回键值
### 7.entries()：返回键值对
### 8.forEach()：使用回调函数遍历每个成员

## 注意：
#### 1.用于数组去重
```
new Set(arr)
```
#### 2.向Set加入值时不会发生类型转换，比较时采用类似 === 的算法比较，区别在于Set中认为NaN等于自身（向Set中加入两个NaN时只能加入一个），=== 中认为NaN!==NaN
#### 3.Set的遍历顺序是插入顺序，使用Set保存一个回调函数列表，调用时就能保证按照加入顺序调用
#### 4.keys()和values()返回一致，因为Set只有键值（或者说键名和键值一样）
#### 5.Set结构的实例默认可遍历，默认生成函数为values，可以直接用for···of···遍历
```
Set.prototype[Symbole.iterator] === Set.prototype.values
```
#### 6.扩展运算符内部使用for···of循环，也可以用来遍历。扩展运算符和Set结构一起可以用作数组去重

##### eg1：
```
let set = new Set(arr);
let result = [···set];
```
##### eg2：
```
let arr = [1,4,2,2,4,5,3];
let result = [···new Set(arr)];
```

#### 7.使用Set实现并集、交集、差集
```
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集 
let union = new Set([...a, ...b]); 
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x))); 
// Set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));  
// Set {1}

```

# Map:
## JS对象，键值对集合（Hash结构）。

## 注意：

### 1.Map构造函数接受数组作为参数
### 2.Set和Map都可以用来生成新的Map
### 3.如果对同一个键多次赋值，后面的值将覆盖前面的值，如果读取一个未知的键值，返回undefined 

```
const map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"

new Map().get('asfddfsasadf')
// undefined
```

#### 4.只有对同一个对象的引用，Map 结构才将其视为同一个键；同理，同样的值的两个实例，在Map结构中被视为两个键

```
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined

```

```
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```
*Map的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（crash）的问题*

#### 5.如果Map的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map将其视为一个键
```
let map = new Map();

map.set(-0, 123);
map.get(+0) // 123

map.set(true, 1);
map.set('true', 2);
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123

```

## Map实例的属性
### 1.map.size：返回Map结构的成员总数
### 2.map.set：设置键名key对应的键值为value，返回Map结构，如果key值已存在，则value会被更新，否则生成该key
*set方法返回的是当前的Map对象，因此可以采用链式写法*

## Map实例的操作和遍历方法
### 1.map.get：读取key对应的键值，找不到返回undefined
### 2.map.has：返回布尔值，表示某个键是否在当前Map对象之中
### 3.map.delete：删除某个键，成功返回true，失败返回false
### 4.map.clear：清除所有成员，无返回值
### 5.keys：返回键名
### 6.values：返回键值
### 7.entries：返回所有成员
### 8.forEach：遍历Map所有成员
```
for (let [key, value] of map.entries()) {
  console.log(key, value);
}
// 等同于
for (let [key, value] of map) {
  console.log(key, value);
}
```
*Map结构的默认遍历器接口为entries方法，即：*
```
map[Symbol.iterator] === map.entries
```
## 与其他数据结构的互相转换
### 1.Map转数组：(···new Map)
### 2.数组转MAp：new Map(arr)
### 3.Map转对象：如果所有 Map 的键都是字符串，它可以无损地转为对象。如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。
```
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
```
### 4.对象转Map：
```
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({yes: true, no: false})
```
### 5.Map转JSON：
#### Map 的键名都是字符串，这时可以选择转为对象 JSON。
```
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)
```
#### Map 的键名有非字符串，这时可以选择转为数组 JSON。
```
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap)
```
### 6.JSON转Map：
#### JSON 转为 Map，正常情况下，所有键名都是字符串。
```
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes": true, "no": false}')
```
#### 有一种特殊情况，整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作。
```
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
```

```
"use strict";
/**
 * 
 * 给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。
 * 你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。
 * 示例:
 * 给定 nums = [2, 7, 11, 15], target = 9
 * 因为 nums[0] + nums[1] = 2 + 7 = 9
 * 所以返回 [0, 1]
 * 给定 nums = [3, 4, 2], target = 6
 * 因为 nums[1] + nums[2] = 4 + 2 = 6
 * 所以返回 [1, 2]
 * 给定 nums = [3, 3], target = 6
 * 因为 nums[0] + nums[1] = 3 + 3 = 6
 * 所以返回 [0, 1]
 */
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
/**
 * 思路：在Map中存储key为target-nums[i]的差值，value为索引i
 * 遍历原始数组nums，如果Map中不存在nums[i]，set差值和i
 * 如果存在nums[i],说明Map中的键nums[i]对应的值和当次循环的i为所求
 */
function twoSum (nums, target) {
    const result = [];
    const myMap = new Map();
    if(nums == null || nums.length <= 1) {
        return false;
    }
    for(let i = 0,len = nums.length; i < len; i++) {
        if(myMap.has(nums[i])) {
            result[0] = myMap.get(nums[i]);
            result[1] = i;
        } else {
            myMap.set(target - nums[i], i);
        }
    }
    console.log(result);
    return result;
}
twoSum([3,3],6);
```