title: 【Vue】使用 Vue2 开发一个项目列表展示应用
date: 2017-04-27 08:50:58
tags: "Vue"
categories: "Vue"
---
## 前言
一直没有找到一个合适的展示个人项目的模板，所以自己动手使用 Vue 写了一个。该模板基于 Markdown 文件进行配置，只需要按一定规则编写 Markdown 文件，然后使用一个 [在线工具](http://project.zhangjikai.com/generator/) 转为 JSON 文件即可。下面是该项目的在线地址和源码。本文主要记录一下项目中用到的相关知识。

[在线演示](http://project.zhangjikai.com/) &nbsp;&nbsp; [源码](https://github.com/zhangjikai/project-list-template)

<!-- more -->
## 效果
程序最终的效果如下图所示：

<img src="/images/project/screenshot.png" style="border: 1px solid #ddd;">

整个项目只包含两个组件：项目介绍 和 侧边导航，逻辑比较简单，十分适合入门。

## 环境配置
这里我们使用 Gulp 和 Webpack 用作项目构建工具。初次使用 Gulp 和 Webpack 可能不太适应，因为它们的配置可能让你看的一头雾水。不过不用担心，这两个毕竟只是一个工具，在初始时没有必要特别的了解它们的工作原理，只要能运行起来就可以。等到使用了一段时间之后，自然而然的就知道该如何配置了。这里主要记录一下项目中使用的配置，如果想要系统的学习如何使用这两个工具，可以参考下面的文章：
* [Gulp入门教程](https://markpop.github.io/2014/09/17/Gulp%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/)
* [一小时包教会 —— webpack 入门指南](http://www.cnblogs.com/vajoy/p/4650467.html)

### Gulp 和 Webpack 集成
Gulp 和 Webpack 集成一个比较简单的方式就是将 Webpack 作为 Gulp 的一个 task，如下面的形式：
```js
var gulp = require("gulp");
var webpack = require("webpack");

gulp.task("webpack", function (callback) {
    //webpack配置文件
    var config = {
        watch: true,
        entry: {
            index: __dirname + '/src/js/index.js'
        },
        output: {
            path: __dirname + '/dist/js',
            filename: '[name].js'
        }
        //........
    };
    webpack(config, function (err, stats) {
        console.log(stats.toString());
    });
});

gulp.task('default', [ 'webpack']);
```
下面我们分别介绍一下 gulp 和 webpack 的配置

### Gulp 配置
Gulp 中主要配置了两个任务：webpack 和 browserSync，这里主要说一下 browserSync。browserSync 主要用来自动刷新浏览器。首先我们配置需要监听的文件，当这些文件发生改变后，调用 browserSync 使浏览器自动刷新页面。下面是具体的配置
```js
var gulp = require("gulp");
var browserSync = require('browser-sync');

// 添加 browserSync 任务
gulp.task('browserSync', function () {
    browserSync({
        server: {
            baseDir: '.'
        },
        port: 80
    })
});

// 配置需要监听的文件，当这些文件发生变化之后
// 将调用 browserSync.reload 使浏览器自动刷新
gulp.task("watch", function () {
    gulp.watch("./**/*.html", browserSync.reload);
    gulp.watch("dist/**/*.js", browserSync.reload);
    gulp.watch("dist/**/*.css", browserSync.reload);
});

// 添加到默认任务
gulp.task('default', ['browserSync', 'watch', 'webpack']);
```

### Webpack 配置
我们使用 webpack 进行资源打包的工作，就是说将各种资源（css、js、图片等）交给 Webpack 进行管理，它会将资源整合压缩，我们在页面中只需引用压缩之后的文件即可。webpack 的基础配置文件如下所示
```js
gulp.task("webpack", function (callback) {

    //webpack配置文件
    var config = {
        // true 表示 监听文件的变化
        watch: true,
        // 加载的插件项
        plugins: [
            new ExtractTextPlugin("../css/[name].css")
        ],
        // 入口文件配置
        entry: {
            index: __dirname + '/src/js/index.js'
        },
        // 输出文件配置
        output: {
            path: __dirname + '/dist/js',
            filename: '[name].js'
        },

        module: {
            // 加载器配置，它告诉 Webpack 每一种文件需要采用什么加载器来处理，
            // 只有配置好了加载器才能处理相关的文件。
            // test 用来测试是什么文件，loader 表示对应的加载器
            loaders: [
                {test: /\.vue$/, loader: 'vue-loader'}
            ]
        },
        resolve: {
            // 模块别名定义，方便后续直接引用别名，无须多写长长的地址
            // 例如下面的示例，使用时只需要写 import Vue from "vue"
            alias: {
                vue: path.join(__dirname, "/node_modules/vue/dist/vue.min.js")
            },
            // 自动扩展文件后缀名，在引入文件时只需写文件名，而不用写后缀
            extensions: ['.js', '.json', '.less', '.vue']
        }
    };
    webpack(config, function (err, stats) {
        console.log(stats.toString());
    });
});
```
webpack 的相关配置说明可以参考前面的给出的文章，下面说一下使用 webpack 2 遇到的坑：

#### extract-text-webpack-plugin

extract-text-webpack-plugin 会将 css 样式打包成一个独立的 css 文件，而不是直接将样式打包到 js 文件中。下面是使用方法
```js
{
    plugins: [new ExtractTextPlugin("../css/[name].css")],
    module: {
        loaders: [{
            test: /\.css$/,
            loader: ExtractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader"
            })
        },
        {
            test: /\.less$/,
            loader: ExtractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader!less-loader"
            })
        }
    },
}
```
这里需要注意的地方就是，extract-text-webpack-plugin 在 webpack 1 和 webapck 2 中的安装方式不同，需要根据使用的 webpack 版本来安装：
```
# for webpack 1
npm install --save-dev extract-text-webpack-plugin
# for webpack 2
npm install --save-dev extract-text-webpack-plugin@beta
```

#### 压缩文件
使用 UglifyJsPlugin 插件可以压缩 css 和 js 文件，但是一开始时总是无法压缩文件，后来查阅了一下资料，大概是因为下面几个原因：
1.  uglifyjs-webpack-plugin 依赖于 uglify-js，而 uglify-js 默认不支持 ES6 语法，所以需要安装支持 ES6 语法的 uglify-js
  ```
  npm install mishoo/UglifyJS2#harmony --save
  ```
2. webpack 2 中，UglifyJsPlugin 默认不压缩 loaders，如果要启动 loaders 压缩，需要加入下面的配置：
  ```js
  plugins: [
    new webpack.LoaderOptionsPlugin({
      minimize: true
    })
  ]
  ```

如果按上面的修改了还是不能压缩文件，可以试着将 node_modules 删除，然后重新安装依赖。

## Vue
本部分主要记录一下程序中用到的 Vue 语法，如果想要系统的学习一下 Vue.js，可以参考下面的文章：
* [Vue.js 教程](https://cn.vuejs.org/v2/guide/)

### HelloWorld
我们首先来看一个最简单的 Vue 示例：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue Demo</title>
</head>
<body>
<script src="https://unpkg.com/vue/dist/vue.js"></script>

<div id="app">
    {{ message }}
</div>

<script>
    var app = new Vue({
        el: '#app',
        data: {
            message: 'Hello Vue!'
        }
    })
</script>

</body>
</html>
```
每个 Vue 应用都会创建一个 Vue 的根实例，在根实例中需要传入 html 标签的 id，用来告诉 Vue 该标签中的内容需要被 Vue 来解析。上面是一个简单的数据绑定的示例，在运行实 &#123;{ message }} 会被解析为 "Hello Vue!"。

### 基础
> 本节参考自 Vue 中文文档，略有修改

在写 Vue 应用之前，我们要熟悉一下 Vue 的基本语法，主要包括数据绑定、事件处理、条件、循环等，下面我们依次看下相关的知识。

#### 数据绑定
Vue.js 使用了基于 HTML 的模版语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。所有 Vue.js 的模板都是合法的 HTML ，所以能被遵循规范的浏览器和 HTML 解析器解析。下面是 Vue.js 数据绑定的相关语法：

* **文本**  
    数据绑定最常见的形式就是使用 "Muestache" 语法（双大括号），如下面的形式：
    ```html
    <span>Message: {{ msg }} </span>
    ```
    Muestache 标签会被解析为对应对象上的 msg 属性值。当 msg 属性发生改变之后，Muestache 标签处解析的内容也会随着更新。

    通过使用 `v-once` 指令，我们可以执行一次性解析，即数据改变时，解析的内容不会随着更新。需要注意的是 `v-once` 会影响该节点上的所有数据绑定
    ```html
    <span v-once>This will never change: {{ msg }}</span>
    ```
* **Raw HTML**  
    不论属性值是什么内容，Muestache 标签里的内容都会被解析为纯文本。如果希望将绑定的值解析为 HTML 格式，就需要使用 `v-html` 指令：
    ```html
    <div v-html="variable"></div>
    ```
* **属性值**  
    Mustache 语法不能用在 HTML 的属性中，如果想为属性绑定变量，需要使用 `v-bind` 指令：
    ```html
    <div v-bind:id="dynamicId"></div>
    ```
    假设 `dynamicId=1`，那么上面代码就会被解析为
    ```html
    <div id="1"></div>
    ```
    另外 `v-bind` 指令可以被缩写为 `:`，所以我们在程序中经常看到的是下面的语法形式：
    ```html
    <div :id="dynamicId"></div>
    <!-- 等价于 -->
    <div v-bind:id="dynamicId"></div>
    ```
* **表达式**
    对于所有的数据绑定， Vue.js 都提供了完全的 JavaScript 表达式支持，如下面的形式：
    ```html
    // 加法
    {{ number + 1 }}

    // 三元表达式
    {{ ok ? 'YES' : 'NO' }}

    // JS 库函数
    {{ message.split('').reverse().join('') }}

    // 指令中使用表达式
    <div v-bind:id="'list-' + id"></div>
    ```

#### 事件处理
通过使用 `v-on` 指令可以监听 DOM 事件来触发 JS 处理函数，下面是一个完整的示例：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue Demo</title>
</head>
<body>
<script src="https://unpkg.com/vue/dist/vue.js"></script>

<div id="app">
    <button v-on:click="increase">增加 1</button>
    <p>这个按钮被点击了 {{ counter }} 次。</p>
</div>

<script>
    var app = new Vue({
        el: '#app',
        data: {
            counter: 0
        },
        methods: {
            increase: function() {
                this.counter++;
            }
        }
    })
</script>

</body>
</html>
```
通常情况下，`v-on` 会被简写为 `@`，所以我们在程序中一般是看到下面的形式
```html
<button @click="increase">增加 1</button>
<!-- 等价于 -->
<button v-on:click="increase">增加 1</button>
```

#### 条件指令 v-if
通过 v-if 指令我们可以根据某些条件来决定是否渲染内容，如下面的形式
```html
<h1 v-if="ok">Yes</h1>
```
我们通常将 v-if 和 v-else 结合起来使用，如下所示：
```html
<div v-if="Math.random() > 0.5">
    Now you see me
</div>
<div v-else>
      Now you don't
</div>
```
在 Vue 2.1.0 中新增了一个 v-else-if 指令，可以进行链式判断：
```html
<div v-if="type === 'A'">
    A
</div>
<div v-else-if="type === 'B'">
      B
</div>
<div v-else-if="type === 'C'">
      C
</div>
<div v-else>
      Not A/B/C
</div>
```

#### 循环指令 v-for
通过 `v-for` 指令，我们可以根据一组数据进行迭代渲染，下面是一个基本示例：
```html
<ul id="example-1">
    <li v-for="item in items">
        {{ item.message }}
    </li>
</ul>
```
```js
var example1 = new Vue({
    el: '#example-1',
    data: {
        items: [
              {message: 'Foo' },
              {message: 'Bar' }
        ]
    }
})
```
上面是一个简单的对数组迭代的示例，我们还可以针对对象进行迭代，如果只使用一个参数，就是针对对象的属性值进行迭代：
```html
<ul id="repeat-object" class="demo">
    <li v-for="value in object">
        {{ value }}
    </li>
</ul>
```
如果传入第二个参数，就是针对对象的属性值以及属性名进行迭代，注意这里二个参数表示的是属性名，也就是 key
```html
<div v-for="(value, key) in object">
    {{ key }} : {{ value }}
</div>
```
如果再传入第三个参数，第三个参数就表示索引
```html
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }} : {{ value }}
</div>
```

### 组件
组件是 Vue.js 最强大的功能之一。组件可以扩展 HTML元素，封装可重用的代码。在我们的程序中包含两个组件：project 组件和 sidebar 组件，如下图所示。这里我们主要介绍单文件组件的使用，即将组件用到 html、js 和 css 都写在一个文件里，每个组件自成一个系统。

<img src="/images/project/screenshot2.png" style="border: 1px solid #ddd;">

#### 文件结构
单文件组件一般使用 ".vue" 作为后缀名，一般的文件结构如下所示：

> project.vue

```html
<template>
    <div>
        {{ key }}
    </div>
</template>

<script>
    export default {
        data: function() {
            return {
                "key": "value"
            }
        },

        methods:  {
            demoMethod: function() {

            }
        }

    }
</script>

<style lang="less">
    @import "xxx.less";
</style>
```
export 将模块输出，default 表明使用文件名作为模块输出名，这就类似于将模块在系统中注册一下，然后其他模块才可用使用 import 引用该模块。

然后我们需要在主文件中注册该组件：

> index.js

```js
import project from '../components/project/project.vue'
Vue.component("project", project);
```

当注册完成之后，就可以 html 中使用该组件了

> index.html

```html
<project></project>
```

#### 生命周期
Vue 的要给组件会经历 创建 -> 编译 -> 挂载 -> 卸载 -> 销毁 等一系列事件，这些事件发生的前后都会触发一个相关的钩子（hook）函数，通过这些钩子函数，我们可以在事件发生的前后做一些操作，下面先看下官方给出的一个 Vue 对象的生命周期图，其中红框内标出的就是对应的钩子函数

![](/images/project/lifecycle.png)

下面是关于这些钩子函数的解释：

| hook | 描述 |
|:-----|:-----|
| beforeCreate | 组件实例刚被创建，组件属性计算之前 |
| created | 组件实例创建完成，属性已绑定，但是 DOM 还未生成， `$el` 属性还不存在 |
| beforeMount | 模板编译/挂载之前 |
| mounted | 模板编译/挂载之后 |
| mounted | 模板编译/挂载之后（不保证组件已在 document 中）|
| beforeUpdate | 组件更新之前 |
| updated | 组件更新之后 |
| activated | for `keep-alive`，组件被激活时调用 |
| deactivated | for `keep-alive`，组件被移除时调用 |
| beforeDestory | 组件销毁前调用 |
| destoryed | 组件销毁后调用 |

下面是钩子函数的使用方法：
```js
export default {
    created: function() {
        console.log("component created");
    },
    data {},
    methods: {}
}
```
#### 父子组件通信
父子组件通信可以使用 props down 和 events up 来描述，父组件通过 props 向下传递数据给子组件，子组件通过 events 给父组件发送消息，下面示意图：

![](/images/project/父子组件.png)
> 图片来自 https://github.com/webplus/blog/issues/10

##### 父组件向子组件传递数据
通过使用 props，父组件可以把数据传递给子组件，这种传递是单向的，当父组件的属性发生变化时，会传递给子组件，但是不会反过来。下面是一个示例

> comp.vue

```html
<template>
    <span>{{ message }}{{ shortMsg }}</span>
</template>

<script>
    export default {
        props: ["message", "shortMsg"],

    }
</script>
```

>index.html

```html
<div id="app">
    <!-- 在这里将信息传递给子组件，:message 表示子组件中的变量名 -->
    <comp :message="hello" :short-msg = "hi"></comp>
</div>

<script>
    var app = new Vue({
        el: '#app',
        data: {
            "hello": "Hello",
            "hi": "Hi"
        }

    })
</script>
```

在上面的流程中，父组件首先将要传递的数据绑定到子组件的属性上，然后子组件在 props 中声明与绑定属性相同的变量名，就可以使用该变量了，需要注意的一点是如果变量采用驼峰的命名方式，在绑定属性时，就要将驼峰格式改为 `-` 连接的形式，如果上面所示 `shortMsg` -> `short-msg`。

##### 子组件向父组件通信
如果子组件需要把信息传递给父组件，可以使用自定义事件：
1. 使用 $on(eventName) 监听事件
2. 使用 $emit(eventName) 触发事件

下面是一个示例：

> comp.vue

```html
<script>
    export default {
        methods: {
            noticeParent: function() {
                // 事件名，传输值
                this.$emit('child_change', "value");
            }
        }
    }
</script>
```

> index.html

```html
<div id="app">
    <comp @child_change="childChange"></comp>
</div>
<script>
    var app = new Vue({
        el: '#app',
        methods: {
            childChange: function(msg) {
                console.log("child change", msg);
            }
        }
    });
</script>
```
在上面的代码中，父组件通过 `v-on` 绑定了 child_chagne 事件，当 child_chagne 事件被触发时候就会调用 childChange 方法。在子组件中可以通过 `$emit` 触发 child\_change 事件。这里需要注意的是事件名不用采用驼峰命名，也不要用 `-` 字符，可以使用下划线 `_` 连接单词。

#### Event Bus 通信
Event Bus 通信模式是一种更加通用的通信方式，它既可以用于父子组件也可以用于非父子组件。它的原理就是使用一个空的 Vue 实例作为中央事件总线，通过自定义事件的监听和触发，来完成通信功能，下面是一个示意图：

![](/images/project/eventbus.jpg)
> 图片来自 https://github.com/webplus/blog/issues/10

下面我们来看一个具体的实例：
* 首先定义一个空的 Vue 实例，作为事件总线
    > EventBus.js

    ```js
    import Vue from 'vue'
    export default new Vue()
    ```
* 在组件一中针对某个事件进行监听
    > comp1.vue

    ```html
    <script>
    import eventBus from "EventBus.js"
    export default {
        created: function() {
            eventBus.$on("change", function() {
                console.log("change");
            })
        }
    }
    </script>
    ```
* 在组件二中触发相应事件完成通信
    > comp2.vue

    ```html
    <script>
    import eventBus from "EventBus.js"
    export default {  
        methods: {
            notice: function() {
                this.$emit('change', "value");
            }
        }
    }
    </script>
    ```

## ES6
> 本节摘自 ECMAScript 6 入门

与 ES5 相比，ES6 提供了更加完善的功能和语法，程序中我们使用部分 ES6 语法，这里做一个简单的记录，如果想要系统的学习 ES6，可以参考下面的文章：
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/)

### let
ES6 新增了 let 命令，用于声明变量。使用 let 声明的变量具有块级作用域，所以在声明变量时，应该使用 let，而不是 var。
```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```
### for of 循环
ES6 借鉴 C++、Java、C# 和 Python 语言，引入了for...of循环，作为遍历所有数据结构的统一的方法
```js
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}
```

### Set 和 Map
ES6 引入了 Set 和 Map 结构。下面是两者的具体介绍

#### Set

> 属性

| 属性 | 描述 |
|:--------|:-------|
| Set.prototype.size | 返回Set实例的成员总数。|

> 方法

| 方法名 | 描述 |
|:-------|:------|
| add(value) | 添加某个值，返回Set结构本身。|
| delete(value) | 删除某个值，返回一个布尔值，表示删除是否成功。|
| has(value) | 返回一个布尔值，表示该值是否为Set的成员。|
| clear() | 清除所有成员，没有返回值。|
| &nbsp; | &nbsp; |
| keys() | 返回键名的遍历器 |
| values() | 返回键值的遍历器 |
| entries() | 返回键值对的遍历器 |
| forEach() | 使用回调函数遍历每个成员 |

使用示例：
```js
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
```

#### Map
> 属性

| 属性 | 描述 |
|:--------|:-------|
| Map.prototype.size | 返回 Map 实例的成员总数。|

> 方法

| 方法名 | 描述 |
|:-------|:------|
| set(key, value) | set方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。|
| get(key) | 读取 key 对应的键值，如果找不到 key，返回 undefined。|
| has(key) | 返回一个布尔值，表示某个键是否在当前 Map 对象之中。|
| delete(key) | 删除某个键，返回true。如果删除失败，返回false。|
| clear() | 清除所有成员，没有返回值。|
| &nbsp; | &nbsp; |
| keys() | 返回键名的遍历器 |
| values() | 返回键值的遍历器 |
| entries() | 返回所有成员的遍历器 |
| forEach() | 遍历 Map 的所有成员。 |

使用示例：
```js
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

## 参考文章
* [Vue 2.0 升（cai）级（keng）之旅](https://discipled.me/posts/troubleshooting-of-upgrading-vue)
* [Vue 2.0开发实践（组件间通讯）](https://github.com/webplus/blog/issues/10)
* [Vuejs2.0 组件通讯总结](http://blog.csdn.net/DeNan_Kong/article/details/68490836)
