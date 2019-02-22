---
title: Vuex源码解读之store
date: 2018-06-05 21:05:37
tags:
- Vuex
categories: 前端框架
about:
---
#### 解读目标
> store对象中有个属性叫state，state包含了全部的应用层级状态。应用中的各个组件如果使用了state，则会保持与同步最新的状态。state就像是Vue中的data，但是state其实是整个Vue应用的data。有个例子：比如说现在有两个非父子关系的子组件a和b，a和b中都监听了state.count,如果a中修改了state.count，那么b中的state.count也相应的会改变。接下来想要探究的问题就是Vuex是如何监听各个组件中的state属性的，其实我觉得有可能又是Object.defineProperty在搞事情——
<!--more-->
#### store对象，在src/store.js目录下
先看**Store类的constructor**，代码很长：
```
export class Store {
    constructor (options = {}) {
    // Auto install if it is not done yet and `window` has `Vue`.
    // To allow users to avoid auto-installation in some cases,
    // this code should be placed here. See #731
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.Vue)
    }

    if (process.env.NODE_ENV !== 'production') {
      assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
      assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
      assert(this instanceof Store, `store must be called with the new operator.`)
    }

    const {
      plugins = [],
      strict = false
    } = options

    // store internal state
    this._committing = false
    this._actions = Object.create(null)
    this._actionSubscribers = []
    this._mutations = Object.create(null)
    this._wrappedGetters = Object.create(null)
    this._modules = new ModuleCollection(options)
    this._modulesNamespaceMap = Object.create(null)
    this._subscribers = []
    this._watcherVM = new Vue()

    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    this.strict = strict

    const state = this._modules.root.state

    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    installModule(this, state, [], this._modules.root)

    // initialize the store vm, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    resetStoreVM(this, state)
  }
}
```
从上往下看吧，首先会使用断言函数assert()对一些条件进行检验，assert()的源码在src/util.js下，实现的就是当不满足某些条件时，会抛出错误。

```
export function assert (condition, msg) {
  if (!condition) throw new Error([vuex] ${msg})
}
```
Store类中的断言函数的作用是：

```
if (process.env.NODE_ENV !== 'production') {
    assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
    assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
    assert(this instanceof Store, `store must be called with the new operator.`)
}
```
1. 确保创建实例前已经在Vue中注册了Vuex，也就是Vue.use(Vuex)。如果在new Vue()之后再去调用Vue.use(Vuex)的话，初始化函数并没有挂载到Vue上，就会导致$store属性无法添加到Vue实例对象上。
2. 确保Promise可以使用，因为Vuex的源码是依赖Promise的。注意要使用Promise，需要添加bable-polyfill的依赖编译。
3. 验证调用方法是否通过new出来的，也就是说判断是否是Store的原型，防止通过直接调用Store()。

接着利用ES6解构赋值拿到的options中的plugins（插件）和strict（是否开启严格模式）。
```
const {
    plugins = [],
    strict = false
} = options
```
接下来主要是创建一些内部的属性：
```
// store internal state
this._committing = false
this._actions = Object.create(null)
this._actionSubscribers = []
this._mutations = Object.create(null)
this._wrappedGetters = Object.create(null)
this._modules = new ModuleCollection(options)
this._modulesNamespaceMap = Object.create(null)
this._subscribers = []
this._watcherVM = new Vue()
```
1. this._committing标志一个提交状态，作用是保证对Vuex中的state的修改只能在mutation的回到函数中，而不能在外部随意修改state
2. this._actions用来存储用户定义的所有actions
3. this._actionSubscribers用来存储所有的actions的订阅者
4. this._mutations用来存储用户定义的所有mutation
5. this._wrappedGetters用来存储用户定义的所有getters
6. this._modules用来存储用户定义的module
7. this._modulesNamespaceMap用来存储用户定义的modules的命名空间
8. this._subscribers用来存储所有对mutation变化的订阅者
9. this._watcherVM是一个Vue对象的实例，主要是利用Vue实例方法$watch来监测变化的

接着就是把Store类的dispatch和commit方法的this指针指向当前store的实例上，dispatch和commit之后再分析。
```
// bind commit and dispatch to self
const store = this
const { dispatch, commit } = this
this.dispatch = function boundDispatch (type, payload) {
    return dispatch.call(store, type, payload)
}
this.commit = function boundCommit (type, payload, options) {
    return commit.call(store, type, payload, options)
}
```
接着往下看，this.strict表示是否开启严格模式，在严格模式下回观测所有的state的变化，建议在开发环境时开启严格模式，线上环境关闭严格模式。
```
// strict mode
this.strict = strict
```
取得当前实例下用户定义的module的根实例下的state，下面就是调用installModule和resetStoreVM这两个方法。
```
const state = this._modules.root.state

// init root module.
// this also recursively registers all sub-modules
// and collects all module getters inside this._wrappedGetters
installModule(this, state, [], this._modules.root)

// initialize the store vm, which is responsible for the reactivity
// (also registers _wrappedGetters as computed properties)
resetStoreVM(this, state)
```
剩下的代码主要是Vuex的初始化的核心。





