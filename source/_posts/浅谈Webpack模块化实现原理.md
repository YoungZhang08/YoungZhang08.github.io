---
title: 浅谈Webpack模块化实现原理
date: 2018-05-23 19:06:12
tags:
- Webpack
categories: 前端构建工具
about:
---
先上总结吧，后面有代码分析：

##### 总结：webpack的模块化原理是酱紫的——首先我们需要在webpack.config.js中取指定入口文件，完了分析webpack编译后的文件可以看到，整个文件由一个自执行函数包裹，入参是一个数组，这个数组包含了所有的模块，函数体的逻辑就是webpack的模块化逻辑。其中有一个函数叫__webpack_require__，参数为moduleId，然后在这个函数体的末尾调用了他传入参数为0并返回，这个0指代的模块就是我们的入口模块。也就是说，从入参的modules数组中取第一个函数进行调用，并传入了module，module.exports以及__webpack_require__这三个参数，然后你可以在入口函数的函数体内看到又调用了__webpack_require__(1)···等等来调用入口文件中所有后续需要的模块，这样就会引入后续的第二个第三个····函数，从第二个函数开始的入参exports，都是这个模块的module.exports通过对象的引用传参，这个模块中需要的其他模块也都是通过__webpack_require__来引的，最终会将module.exports return出来。

<!--more-->

webpack(1.x)编译后的文件：

```
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId])
/******/ 			return installedModules[moduleId].exports;
/******/
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			exports: {},
/******/ 			id: moduleId,
/******/ 			loaded: false
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.loaded = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, exports, __webpack_require__) {

	(function(root) {
	
	    __webpack_require__(1);
	    __webpack_require__(5);
	    __webpack_require__(7);
	
	    var homepage = __webpack_require__(67);
	    var homepageWiki = __webpack_require__(73);
	    var homepageSetting = __webpack_require__(75);
	    Vue.use(VueRouter);
	    var App = Vue.extend({});
	    var router = new VueRouter();
	    root.router = router;
	    router.map({
	        '/': {
	            component: login
	        },
	        '/login': {
	            name: 'login',
	            component: login
	        }
	   });
	   router.start(App, '#app');
	})(window);
/***/ }),
/* 70 */
/***/ (function(module, exports, __webpack_require__) {
	var getHeaders = __webpack_require__(71);
	var baseAjax = function(api, data, callback) {
	    var headers = getHeaders();
	    // console.log(api, '')
	    // console.log(headers);
	    $.ajax({
	        url: api,
	        type: 'POST',
	        dataType: 'json',
	        data: data,
	        success: callback,
	        headers: headers
	    });
	};
	module.exports = baseAjax;
/***/ }),
/* 77 */
/***/ (function(module, exports, __webpack_require__) {
	var baseAjax = __webpack_require__(70);
	module.exports = {
	}
/***/ }),
/******/ ]);
```

webpack.config.js文件：

```
var webpack = require('webpack');
var ExtractTextPlugin = require('extract-text-webpack-plugin');//将所有css文件导成一个
var extractLESS = new ExtractTextPlugin('./css/[name].less');
var extractCSS = new ExtractTextPlugin('./css/[name].css');
// var proxy = require('http-proxy-middleware');

module.exports = {
	entry : __dirname + '/src/app.js',
	output : {
		path : __dirname + '/lastpack',
		filename : '[name].pack.js'
	},
	module : {
		loaders : [
			{
				test : /\.html$/,
				loader : 'html'
			},
			{
				test : /\.(css|less)/,
				loader : ExtractTextPlugin.extract('style', 'css!less'),
				exclude: '/lastpack/'
			}
		]
	},
	externals : {
		'vue' : 'Vue',
		'vue-router' : 'VueRouter' //在项目中require的类库不会被合并到出口文件
	},
	plugins : [
		extractCSS,
		new webpack.optimize.UglifyJsPlugin({// webpack内置插件压缩JS/CSS
			compress : {
				warnings : false
			}
		})
	],
	devServer: {
		proxy: {
			'/api/*': {
				target: 'https://localhost',
				changeOrigin: true
			}
		}
	}
};
```

精简一下代码：

```
(function(modules){
})([]);
```
这块的自执行函数的入参是个数组，这个数组包含了所有模块，这些模块都包裹在函数中。自执行函数体里的逻辑就是处理模块的逻辑，关键就在__webpack_require__这个函数，这个函数就是require的替代。自执行函数的函数体里先定义了这个函数，然后调用了他并传入一个moduleId，这个例子中是0，也就是我们的入口模块__dirname + '/src/app.js'的内容。

__webpack_require__内部执行了

```
// Execute the module function
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
// Flag the module as loaded
module.loaded = true;
// Return the exports of the module
return module.exports;
```
也就是从入参的modules数组中取第一个函数进行调用，并传入了module，module.exports以及__webpack_require__，再看第一个函数（入口函数）的逻辑：

```
function(module, exports, __webpack_require__) {

	(function(root) {
	
	    __webpack_require__(1);
	    __webpack_require__(5);
	    __webpack_require__(7);
	    ···
	});
}
```
可以看到入口模块又调用了__webpack_require__(1)去引入第二个函数。

这里入参的exports就是这个模块的module.exports通过对象的引用传参，最后会将module.exports return出来。到这里就完成了__webpack_require__的使命。

比如说在入口模块中又调用了__webpack_require__(1)，就会得到这个模块返回的module.exports。
