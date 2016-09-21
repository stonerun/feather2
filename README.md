feather2
==========================

![](https://img.shields.io/npm/v/feather2.svg) ![](https://img.shields.io/npm/dm/feather2.svg)

feather2.0是继[feather](http://github.com/feather-team/feather)之后基于fis3.0进行扩展的工程化框架。

2.0的架构做出了很大的调整，提高用户的易用性，并于feather1.x不同，2.0仅仅只适用于纯静态页面的前端项目，比如webapp，或结合一些mvvm框架进行开发的项目。

基于2.0可以非常容易的再次扩展出动态语言的工程化框架，并且开发量也较少，如: [lothar](http://github.com/feather-team/lothar)(blade模板引擎)

####2.0与1.x部分功能比较：

| 功能                  | 1.x               | 2.0   |
|-----------------------|------------------:|------:|
|fis基本功能   | 支持             |支持     |
|本地服务器、url转发、mock数据   | 支持（java）             |支持（node）     |
|压缩、合并、csssprite、预编译   | 支持             |支持     |
|项目脚手架   | 支持             |支持     |
|livereload   | 支持             |支持     |
|模块化   | 支持             |支持     |
|模板继承   | 不支持             |支持     |
|bigrender/pipe/quickly   | 部分支持             |支持     |
|包管理   | 不支持             |支持     |
|多人协同   | 支持             |支持     |
|多模块开发   |动态支持             |动态支持     |
|静态资源位置优化、去重   | 支持            |支持     |
|静态资源按需加载、combo   | 不支持             |支持     |
|远程deploy方式   | http             |http/ftp     |

## 使用文档

### gogogo

#### 安装
```sh
npm install -g feather2
```

#### 使用脚手架创建项目
```sh
feather2 init demo
```

#### 编译项目
```sh
feather2 release -r demo
```

#### 开启本地server，预览项目
```sh
feather2 server start
```

### 2.0的目录结构规范
```sh
├── components      #组件包
│   ├── backbone
│   │   ├── backbone.js
│   │   └── bower.json
│   └── underscore
│       ├── bower.json
│       └── underscore.js
├── conf
│   ├── conf.js     #2.0的核心配置文件
│   ├── deploy
│   │   └── local.js
│   ├── pack.json   #打包文件，对全局起作用
│   └── rewrite.js  #url重写文件
├── data            #测试目录，同1.x中的test目录
│   └── ajax
│       └── test.json
├── index.html
├── layout.html
├── pagelet         #pagelet目录，feather会对此目录中的文件进行特殊封装，此文件夹下的文件同widget
│   ├── ajax
│   │   ├── ajax.css
│   │   └── ajax.html
│   └── bigrender
│       ├── bigrender.css
│       ├── bigrender.html
│       └── bigrender.js
├── static
│   ├── index.css
│   └── index.js
├── test
│   └── test.html   #预览情况下才会产出，正式环境下不会产出，此目录用于测试，预览下于其他模板文件无异
└── widget
    ├── footer
    │   ├── footer.css
    │   └── footer.html
    └── header
        ├── header.css
        ├── header.html
        └── header.js
```

#### page
page目录在feather2.0中并未被脚手架创建，所以在2.0中，任意非feather占用的目录都可作为page目录使用，page级别文件则表示是一个标准正常可访问的页面，比如index.html， 通过浏览器可直接访问，同时，只有page文件可用于被继承。

#### widget
widget目录存放一些页面和模版相关的小部件，比如公用头部，公用尾部，sidebar，nav等。

需要注意的是，为了提高开发者使用的方便，widget中的html模版，会自动加载同名js及css，js以require.async的方式引入，而css以link方式引用，无需开发者自行引入，当然也可自行引入。

#### comonents
components目录存放一些js部件和模块， 可结合fis的__inline语法，进行js渲染html的工作，比如webapp中会经常用到。同时，为了提高开发者体验，加载components中的js时，会自动加载对应js同名的css文件。比如

```js
require.async('a');
```

则会加载components/a/a.js，同时也会使用异步加载方式加载components/a/a.css，并且feather会自动处理好所有依赖问题，开发者仅需要关注所需要引入的components即可。

#### pagelet
pagelet目录同widget目录，不同的是，pagelet可以作为异步加载请求返回， feather会自动对其做对应的封装，并处理依赖。同时也可直接使用 <pagelet> 标签直接引入， 以做bigrender方式的优化

### 组件包（components）
2.0中直接对bower install进行了支持，因为bower的资源是非常丰富的，甚至可以支持github上的某一个仓库的分支。

但是bower包几乎都采用了amd或umd规范，甚至有些bower包的是没有规范的，而feather因为静态资源管理的问题，采用了commonjs规范，因此feather对其进行了一定的处理，尽可能的让bower中的包能够正常的运行。同时删除了bower包中无用的文件。

另外，在项目中支持直接在页面中引用某一个组件包，而不需要知道具体引用的包文件，就像npm结合node中的require一样，大大提高开发者的便捷

#### 安装包
```sh
cd demo
feather2 install socket.io-client bootstrap vue #包会被直接安装至components目录下
```

#### 使用包

index.html

```html 
<link href="bootstrap/css/bootstrap.css" type="text/css" />

<script>
require.async('vue', function(Vue){
    console.log(Vue);
});

require.async('socket.io-client', function(io){
    console.log(io);
});
</script>
```

#### 包查找原理及规则
feather首先会对所有components目录中包当中的**.json文件进行分析，提取main属性，并缓存。
当使用某一个文件时，无论是否是全路径还是段路径，都会直接先查找components中的包，如果找到正确的文件，则引用，否则会直接按照全路径做对应的处理

### 模板语法标签
2.0考虑到了单或多页面应用，及为了提高页面的重用性，对html进行了扩展，支持模板继承(extends, block)，模板引用功能(widget, pagelet)，并可互相组合使用。

#### extends、block、widget

layout.html
```html
<html>
<head>
<title>
    <!--可重写title-->
    <block "title">layout.html</block>
</title>
</head>
<body>

<!--引用widget/a.html-->
<widget "a">

<!--声明一个block块，用于被重写-->
<block "content">
<!--此处可以无内容-->
lalala
</block>

</body>
</html>
```
main.html
```html
<!--继承layout.html-->
<extends "./layout.html">

<block "title">main.html</block>
<block "content">lalala, i'm main.html</block>

<div>因为页面是继承，所以我会被feather忽略掉，不显示</div>
<block "ignore">
    lalalala
</block>
```

#### pagelet

使用格式

```html
<pagelet "pagelet名[#外部包裹textarea的id]">
```

main.html
```html
<!--继承layout.html-->
<extends "./layout.html">

<block "title">main.html</block>
<block "content">
    <div id="show"></div>

    <pagelet "a#lalala">
    
    <scirpt>
    require.async('jquery', function($){
        setTimeout(function(){
            //5秒后，将lalala里的东西，显示在show容器中
            $('#show').append($('#lalala').val());
        }, 5000);
    });
    </script>
</block>
```

相同布局的页面可使用extends结合block的方式进行开发， 一些通用模块可使用widget引入。

