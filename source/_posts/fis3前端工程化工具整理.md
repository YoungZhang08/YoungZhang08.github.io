---
title: fis3前端工程化工具整理
date: 2018-05-28 21:12:46
tags: 
- fis3
categories: 前端构建工具
about:
---

### fis3的工作原理：
> fis3是真正地从前端工程化的角度出发，不单单是个打包工具，它基于文件对象进行构建，每个进入fis3的文件都会被实例化为一个file对象，整个构建流程都对这个对象进行操作完成构建任务。fis3没有入口文件的概念，是根据内置打包阶段生成的资源表定义之后的产出规则。

### fis3的构建流程：
> fis3将构建流程分为了单文件编译和打包两个阶段。单文件编译又分为了lint、parser、preprocessor、standard、postprocessor、optimizer六个阶段。打包分为prepackager、packager、spriter、postpackager四个阶段。
<!--more-->

```
fis.release = function (opt) {
  var src = fis.util.find(fis.project.root);
  var files = {};
  src.forEach(function (f) {
    var file = new File(f);
    files[file.subpath] = fis.compile(file);
  });
  var packager = fis.matchRules('::package');
  var keys = Object.keys(packager);
  var ret = {
    files: files,
    map: {}
  };
  if (packager.indexOf('prepackager') > -1) {
    pipe('prepackager', ret);
  }

  if (packager.indexOf('packager') > -1) {
    pipe('packager', ret);
  }

  files.forEach(function (f) {
    // 打包阶段产出的 map 表替换到文件
    if (f._isResourceMap) {
      f._content = f._content.replace(/\b__RESOURCE_MAP__\b/g, JSON.stringify(ret.map));
    }
  });

  if (packager.indexOf('spriter') > -1) {
    pipe('spriter', ret);
  }
  if (packager.indexOf('postpackager') > -1) {
    pipe('postpackager', ret);
  }
}
```

> 1. 扫描项目目录拿到文件并初始化出一个文件对象列表
> 2. 对文件对象中每一个文件进行单文件编译
> 3. 获取用户设置的package插件，进行打包处理（包括合并图片）

![](https://note.youdao.com/yws/public/resource/977ae713311b47c735d17d0a22726d2a/xmlnote/WEBRESOURCEfbb003f65a629ea378998ad3e27b9108/4523)

单文件编译处理了六个插件扩展点，通过用户配置启用某些插件：
> 1. lint 代码校验检查，比较特殊，需要release命令命令行添加-l参数
> 2. parser 预处理阶段，比如less、sass、es6、react前端模板等都在此处预编译处理
> 3. preprocessor 标准化前处理插件
> 4. standard 标准化插件，处理内直插件
> 5. postprocessor 标准化后处理插件
> 6. optimizer 代码优化


打包处理了四个扩展点，通过用户配置启用某些插件：
> 1. prepackager 打包前处理插件扩展点
> 2. packager 打包插件扩展点，通过此插件收集文件依赖信息、合并信息产出静态资源映射表
> 3. spriter 图片合并扩展点，如csssprites
> 4. postpackager 打包后处理插件扩展点

### fis3的配置文件
> 通过glob语法match到某些文件，然后对这些文件定义各自的各个阶段的插件处理、发布规则，还能通过media定义不同环境的发布规则。感觉fis3的配置更优雅，可操作性更强。文件在什么阶段使用什么插件在这个阶段发生什么事情都可以很清楚地看到。

```
/**
 * 【模块划分】
 * 一、特殊文件处理
 * 二、预处理es6+css/less+vue
 * 三、模块化处理
 * 四、优化optimizer
 * 五、media多环境
 */

fis.require('smarty')(fis);
var namespace = 'cloan';
fis.set('namespace', namespace);
fis.set('templates', '/template/' + namespace);
fis.set('statics', '/static/' + namespace);

/**
 * 一、特殊文件处理
 */

var ignoreList = [
    '*.{md, sh, idea, DS_Store}',
    'BCLOUD',
    'BCLOUD.qa',
    'fis-conf.js',
    'component.json',
    '**/example/*',
    '.git/**'
].join(',');
// 1. 忽略release文件
fis.match('{' + ignoreList + '}', {
    release: false
})
// 2. test文件放到根目录
    .match('/app/test/(**)', {
        release: '/test/${namespace}/app/$1'
    })
    .match('/test/(**)', {
        release: '/test/${namespace}/$1'
    })
    // 3. /app/static/config/api.js文件里面${appName}替换
    .match('**/api**.js', {
        postprocessor: function (content, file, settings) {
            return content.replace(/\$\{appName\}/g, namespace);
        }
    })
    // 4. 指定tpl为page(与默认不一样)
    .match('**.tpl', {
        release: '${templates}/$0',
        url: '${namespace}/$0',
        extras: {
            isPage: true
        }
    });

/**
 * 二、预处理es6+css/less+vue
 */
var vueList = [
    'app/component/**.vue',
    'components/**.vue',
    'loan-components/**vue'
].join(',');

// todo: 可以优化为所有的js?
var es6List = [
    '**.vue:js',
    'app/**.tpl:js',
    'app/page/**.js',
    'app/component/**.js',
    'app/static/config/**.js',
    'static/js/util/**.js',
    'components/**.js',
    'loan-components/**.js',
    'widget/biz/error-reporter/index.js'
].join(',');

var lessList = [
    '*.less',
    'components/**.vue:less',
    'app/component/**.vue:less',
    'loan-components/**.vue:less',
    'page/activity/**.css'
].join(',');

// 1. vue处理
fis.match('{' + vueList + '}', {
    isMod: true, // todo:待确定
    rExt: 'js',
    useSameNameRequire: true, // todo: 待确定
    parser: fis.plugin('vue-component', {
        cssScopeFlag: 'vue-c'
    })
})
// 2. es6 处理
    .match('{' + es6List + '}', {
        parser: [
            fis.plugin('babel-6.x', {
                plugins: [
                    'add-module-exports',
                    'transform-object-assign',
                    'async-to-promises',
                    'array-includes'
                ]
            })
        ],
        rExt: 'js'
    })
    // 3. css/less 处理
    .match('{' + lessList + '}', {
        parser: [
            fis.plugin('less-2.x'),
            fis.plugin('rem', {
                rem: 37.5,
                dpr: 1,
                fontSize2Rem: true
            })
        ],
        rExt: '.css'
    })
    // 加前缀。todo: 是否可以去掉less
    .match('*.{css,less}', {
        preprocessor: fis.plugin('cssprefixer', {
            browsers: ['Chrome > 1'],
            cascade: true
        }),
        useSprite: true
    })
    .match('*.{png, jpg, svg, gif}', {
        useHash: true,
        useSprite: true,
        optimizer: null
    });

/**
 * 三、模块化处理
 */
// 1. path短路径
fis.hook('commonjs', {
    // 要加vue
    extList: [
        '.js', '.vue'
    ],
    paths: {
        api: '/app/static/config/api.js',
        constant: '/app/static/config/constant.js',
        notifyNativeError: '/app/static/config/notify-native-error/index.js',
        formConfig: '/app/static/config/form/index.js',
        component: '/app/component'
    }
})
// 2. node_modules
    .hook('node_modules');

// 3. js模块化处理
var modList = [
    '/node_modules/**.js', // 因为某些js没有模块化
    '/app/static/config/**.js',
    '/app/static/js/**.js',
    '/app/page/**.js',
    '/app/component/**.js',
    '/components/**.js',
    '/loan-components/**.js'
].join(',');

fis.match('{' + modList + '}', {
    isMod: true,
    useHash: true
})
    .match(/^\/page\/activity\/activity-static\/track\/.*\.(js)/i, {
        isMod: false
    })
    .match('/components/fin-fg/error-reporter/index.js', {
        isMod: false
    });

/**
 * 四、优化optimizer
 */
// 1. 压缩：
// 【默认没有压缩】
// - 组件vue转换成的js
// - png
// 【默认有压缩】
// - tpl
// - css
fis.match('*.{js, vue}', {
    optimizer: fis.plugin('uglify-js', {
        output: {
            // smarty 超过200字符会截断
            /* eslint-disable */
            max_line_len: 200
            /* eslint-enable */
        }
    })
})
    .match('*.png', {
        optimizer: fis.plugin('png-compressor', {})
    });

/**
 * 五、media多环境
 */

// 本地环境
// 开发模式
fis.media('debug')
    .match('*.{js, vue, less, css, png, jpg, svg}', {
        useHash: false,
        useSprite: false,
        optimizer: null
    });
// 模拟线上
fis.media('devonline')
    .match('::packager', {
        packager: fis.plugin('deps-pack', pkgMap)
    });
// 2. 联调环境

/**
 * 不能同步到rd机器检查点
 *
 * 1. less 或者 css相关文件没有文件注释，或者只有根选择器没有内容
 * 2. receiver.php没有部署或者删除
 *
 */

// 联调推到rd机器
var rdInfo = [
    {
        name: 'rd',
        path: '/home/work/odp-cloan',
        receiver: 'http://gzns-nbg-fpu-aug-c02xi2-87.gzns.baidu.com:8080/umoneyweb/receiver.php'
    },
    {
        name: 'rd3',
        path: '/home/work/odp-cloan3',
        receiver: 'http://gzns-nbg-fpu-aug-c02xi2-87.gzns.baidu.com:8080/umoneyweb/receiver.php'
    },
    {
        name: 'dev',
        path: '/home/work/odp',
        receiver: 'http://gzns-um-c63.epc.baidu.com:9999/receiver.php'
    }
];
// 3. 测试环境+线上环境
onlineEnv('qa');
onlineEnv('prod');

function onlineEnv(mediaName) {
    fis.media(mediaName)
        .match('**/icon/**', {
            release: '/static/${namespace}/pkg/font/$1'
        })
        .match('*.{png, jpg, gif, svg}', {
            release: '/static/${namespace}/pkg/image/$0'
        })
        .match('{components/**.js, app/page/layout.tpl:js, loan-components/**.js}', {
            parser: [
                fis.plugin('jdists', {
                    remove: 'debug,test,dev'
                }),
                fis.plugin('babel-6.x', {
                    plugins: [
                        'add-module-exports',
                        'transform-object-assign',
                        'async-to-promises',
                        'array-includes'
                    ]
                })
            ]
        })
        .match('::packager', {
            packager: fis.plugin('deps-pack', pkgMap)
        })
        .match('**', {
            deploy: [
                fis.plugin('skip-packed'),
                fis.plugin('local-deliver', {
                    to: fis.get('options.dest') || fis.get('options.d') || './output'
                })
            ]
        });

    if (mediaName === 'prod') {
        fis.media(mediaName)
            .match('!*.tpl', {
                domain: ['https://umoney.bdstatic.com/webroot']
            });
    }
}
```
### fis3的输出
> fis3的打包阶段只搜集所有文件列表以及各种依赖关系，根据打包阶段生成的资源表自定义产出规则。

### fis3可以提供假mock数据
fis3提供mock假数据模拟协助前端开发，需要放到服务器的/mock/sample.json目录下，确保通过http://127.0.0.1:8080/mock/sample.json可以访问到

```
{
 "error": 0,
 "message": "ok",
 "data": {
   "uname": "foo",
   "uid": 1
 }
}
```
准备一个server.conf配置文件放在服务器的/mock/server.conf目录，内容如下：

```
rewrite ^\/api\/user$ /mock/sample.json
```
