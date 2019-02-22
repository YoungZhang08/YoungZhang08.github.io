---
title: Vuex源码解读之install
date: 2018-06-05 21:04:10
tags:
- Vuex
categories: 前端框架
about: 
---

#### 解读目标
> 在根实例中注册store选项，该store实例会注入到格努组件下面的所有子组件中，并且子组件都能通过this.$store访问，那为啥它们都可以通过this.$store就能访问到store选项？从入口文件开始——

<!--more-->
#### 入口文件，在src/index.js目录下
```
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```
可以看到这里就是vuex暴露出来的API，首先来看看install这个公开方法，当在vue中使用插件时，必须使用install方法。所以我们一般都是这样使用vuex的：

```
import Vue from 'vue'
import Vuex from 'vuex'
...
Vue.use(Vuex)
```
因为vuex也可以看成是一个插件，所以在调用vue.use(Vuex)的时候，实际上调用了install的方法并传入了Vue的引用。

#### install方法，在src/store.js目录下

```
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```
对Vue的判断主要是保证install方法只执行一次，这里把install方法的参数在最后赋值给了Vue变量，这样的话在index.js文件的其他地方也可以使用Vue这个变量。install方法的最后调用了applyMixin方法：

#### applyMixin方法，在src/mixin.js目录下：

```
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```
这段代码实现了全局注册混合对象。先读取了Vue的版本对其做判断，如果版本是2.x，则在beforeCreate这个生命周期中调用vuexInit方法进行初始化，如果版本是2.x一下，则在init这个生命周期中调用vuexInit方法进行初始化。vuexInit方法中，实际做的事情就是组件取得Vue对象的$options，把$options里的store赋值给$store。如果$options没有store，则向其parent向上查找并赋值。这也就是为什么我们在Vue组件中可以通过this.$store.xxx访问到Vuex的各种数据和状态。



