---
title: 解锁js数组去重的多种方法
date: 2018-06-01 14:54:08
tags:
- 数组去重
categories: JS手撕
about:
---

方法一：双重循环+新建数组(1)

思路：新建一个数组存放返回的去重后的数组，设置一个标志来标识原数组和新数组中的元素是否重复，外层循环控制趟数，内层循环控制新数组和原数组元素的比较，如果新数组和原数组的元素相同，标志位置true，并跳出当前循环，如果新数组和原数组元素不同，执行完当前循环后，将外层循环变量指向的元素push进新数组

<!--more-->

```
function unique(arr) {
    const res = [];
    let isRepeat;

    for(let i = 0, len = arr.length; i < len; i++) {
        isRepeat = false;
        for(let j = 0, resLen = res.length; j < resLen; j++) {
            if(arr[i] === res[j]) {
                isRepeat = true;
                break;
            }
        }
        if(!isRepeat) {
            res.push(arr[i]);
        }
    }

    console.log(res);
    return res;
}
```

方法二：双重循环+新建数组(2)

思路：新建一个数组存放返回的去重后的数组，设置一个标志来标识原数组和新数组中的元素是否重复，外层控制趟数，内层比较，如果外层循环指向的元素和内层循环指向的元素相同，将标志位置true，并且跳出当前循环，但是不往新数组push外层循环变量指向的元素，继续下一趟循环，直到外层循环指向的变量和内层循环循环完之后都没有遇到相同元素，则把外层循环变量指向的元素push进新数组
```
function unique(arr) {
    const res = [];
    const len = arr.length;
    let isRepeat;

    for(let i = 0; i < len; i++) {
        isRepeat = false;
        for(let j = i + 1; j < len; j++) {
            if(arr[i] === arr[j]) {
                isRepeat = true;
                break;
            }     
        }
        if(!isRepeat) {
            res.push(arr[i]);
        }
    }
    console.log(res);
    return res;
}
```

方法三：双重循环+新建数组(3)

思路：新建一个数组存放返回的去重后的数组，外层循环
```
function unique(arr) {
    const res = [];
    const len = arr.length;

    for(let i = 0; i < len; i++) {
        for(let j = i + 1; j < len; j++) {
            if(arr[i] === arr[j]) {
                j = ++i;
            }
        }
        res.push(arr[i]);
    }

    console.log(res);
    return res;
}
```

方法四：新建数组+indexOf()

思路：forEach遍历数组元素，利用indexOf()返回数组元素的索引来判断新数组中是否有当前遍历的元素的索引，如果没有的话，直接将当前元素push进新数组
```
function unique(arr) {
    const res = [];
    
    arr.forEach(item => {
        if(res.indexOf(item) === -1) {
            res.push(item);
        }
    });

    console.log(res);
    return res;
}
```

方法五：新建数组+filter+indexOf()

思路：利用filter过滤功能简化外层循环，利用indexOf返回数组中元素第一次出现的位置，如果和filter过滤的索引相同的话就把当前元素push进新数组，如果不同就被过滤掉
```
function unique(arr) {
    var res = [];

    arr.filter((item, index) => {
        if(arr.indexOf(item) === index) {
            res.push(item);
        }
    });

    console.log(res);
    return res;
}
```

方法六：新建数组+排序+单层循环

思路：对要去重的数组先进行排序，然后新建数组用来存放去重后的数组，在原数数组中遍历和新数组每一次的最后一个元素去比较，如果当前遍历的元素和新数组的最后一个元素不同，把当前遍历的元素push进新数组
```
function unique(arr) {
    const res = [];
    arr.sort();

    for(let i = 0, len = arr.length; i < len; i++) {
        if(arr[i] !== res[res.length - 1]) {
            res.push(arr[i]);
        }
    }

    console.log(res);
    return res;
}
```

方法七：新建数组+排序+单层循环

思路：对要去重的数组先进行排序，然后新建数组用来存放去重后的数组，循环遍历原数组，如果第i项和第i+1项元素不同的话，将第i项push进新数组，如果相同的话进行下一次循环
```
function unique(arr) {
    const res = [];
    arr.sort();

    for(let i = 0, len = arr.length; i < len; i++) {
        if(arr[i] !== arr[i+1]) {
            res.push(arr[i]);
        }
    }

    console.log(res);
    return res;
}
```

方法八：新建数组+单层循环+includes()

思路：新建数组用来存放去重后的数组，forEach遍历数组，如果新数组中不包含当前遍历的元素，就把当前遍历的元素push进新数组

注：includes用来判断一个数组中是否包含一个指定项，返回值为布尔值
```
function unique(arr) {
    const res = [];

    arr.forEach(item => {
        if(!res.includes(item)) {
            res.push(item);
        }
    });

    console.log(res);
    return res;
}
```

方法九：排序+reduce()

思路：reduce() 方法对数组中的每个元素（从左到右）执行回调，将其减少为单个值。回调函数的两个参数，init作为回调的第一个调用的第一个参数的值，currentValue为数组中正在处理的当前元素
```
function unique(arr) {

    console.log(arr.sort().reduce((init, currentValue) => {
        if(init.length === 0 || init[init.length - 1] !== currentValue) {
            init.push(currentValue);
        }

        return init;
    }, [])); 
}
```

方法十：Object键值对

思路：利用对象的key不可以重复。但是这里要注意三个问题：

1.无法区分隐式转换成字符串后一样的值，比如1和‘1’

2.无法处理复杂数据类型，比如对象（因为对象作为key会变为[object objec]）

3.特殊数据，比如‘proto’，因为对象的proto属性无法被重写

```
function unique(arr) {
    const res = [];
    const obj = {};

    for(let i = 0, len = arr.length; i < len; i++) {
        // 使用JSON.stringify()进行序列化
        if(!obj[typeof arr[i] + JSON.stringify(arr[i])]) {
            // 将对象序列化之后作为key来使用
            obj[typeof arr[i] + JSON.stringify(arr[i])] = true;
            res.push(arr[i]);
        }
    }

    console.log(res);
    return res;
}
```
方法十一：ES6 Map(1)
```
function unique(arr) {
    const res = [];
    const temp = new Map();

    for(let i = 0, len = arr.length; i < len; i++) {
        if(!temp.get(arr[i])) {
            temp.set(arr[i], 1);
            res.push(arr[i]);
        }
    }

    console.log(res);
    return res;
}
```

方法十二：ES6 Map(2)
```
function unique(arr) {
    const temp = new Map();

    console.log(arr.filter(item => {
        return !temp.has(item) && temp.set(item, 1);
    }));
}
```

方法十三：ES6 Set(1)
```
function unique(arr) {
    console.log(Array.from(new Set(arr)))
    return Array.from(new Set(arr));
}
```

方法十四：ES6 Set(2)

```
const unique = (arr) => [...new Set(arr)]
```
