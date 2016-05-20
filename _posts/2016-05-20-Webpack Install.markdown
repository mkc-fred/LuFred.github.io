---
layout: post
title: "WebPack初体验"
subtitle: "最近公司新的项目在前端换了一套新的一套View框架([React](https://facebook.github.io/react/)),所以原来那套Knockout＋requiseJS技术组合有点不适合用在这里，Google了一下发现一个用于React开发和模块管理的最佳工具是一个叫Webpack的货。所以下面就是对它的一个最基本了解。
date: 2016-05-20
author: LuJiangBo
category: JavaScript
tags: JavaScript
finished: true
---


//webpack 安装及基本使用
##webpack是什么
[Webpack](http://webpack.github.io)是当下热门的前端资源模块化管理和打包工具，它可以将松散的模块按照依赖规则打包成符合生产环节部署的前段资源。还可以将按需加载的模块进行代码分隔，等到实际需要的时候在异步加载，通过loader的转换，任何形式的资源都可以视作模块，比如CommonJs模块、AMD模块、CMD模块、ES6模块、CSS、image、JSON、LESS等。
##安装
1.安装node.js  
&emsp;&emsp;node附带一个npm的包管理工具,[下载地址](https://nodejs.org/en/)

2.使用npm安装webpack  
&emsp;&emsp;命令:$ npm install webpack -g  -g参数表示webpack全局安装

3.在项目中使用webpack  
&emsp;&emsp;命令:$ npm init   
>该命令会出创建一个名为package.json的文件，用来记录你项目中依赖的包

4.如果不想把你的项目上传到npm，你可以把webpack添加到package.json  
&emsp;&emsp;$ npm install webpack --save-dev    
>--save-dev表示把webpack安装到当前目录的node_modules文件夹下        

5.安装特定版本的webpack
&emsp;&emsp;$ npm install webpack@1.2.x --save-dev



##使用
>下面是一个最简单的使用webpack的一个例子

####1.创建一个cats.js 文件

{% highlight javascript %}
    var cats = ['dave', 'henry', 'martha'];
    module.exports = cats;
{% endhighlight %}

####2.创建一个app.js 文件 依赖于cats.js

{% highlight javascript %}
    cats = require('./cats.js');
    console.log(cats);
{% endhighlight %}

####3.使用webpack命令打包app.js  
命令:webpack ./app.js app.bundle.js
>会创建生产一个打包后的app.bundle.js文件

####4.也可以创建一个webpack.config.js文件，替代第三步webpack后面的参数
在webpack.config.js中添加如下代码:
{% highlight javascript %}
    const webpack = require('webpack');
    module.exports = {
        entry: './src/app.js',              //需要被打包的文件
        output: {
            path: './bin',                  //文件打包后的输出路径
            filename: 'app.bundle.js',      //输出文件名多个文件时也可使用'[]',批量命名'[name].bundle.js'
        }
    }
{% endhighlight %}  

####5.config中可以通过module加载不同功能的模块解析器，比如解析React的jsx文件
//多个loader用!分隔 loader:'babel!jsx'
首先得在项目目录安装babel-loader
命令:npm i --save-dev babel-core babel-loader  
其次在config中配置module参数  
% highlight javascript %}
        module: {
            loaders: [{
                test: /\.jsx?$/,
                exclude: /node_modules/,
                loader: 'babel',
            }]
        }
{% endhighlight %}  

####6 插件依赖
通常我们还需要额外的包来支持我们的项目，比如js的压缩，公共js代码的提取。
这时候我们可以在config文件中通过配置plugins来添加相应功能的插件包
如：
% highlight javascript %}
 plugins: [
        new webpack.optimize.UglifyJsPlugin({                   //js压缩插件
            compress: {
                warnings: false,
            },
            output: {
                comments: false,
            },
        }),
        new webpack.optimize.CommonsChunkPlugin('common.js');   //公共js代码的提起，创建一个common.js文件
    ]
{% endhighlight %} 

##番外  
$ npm install webpack-dev-server --save-dev
>ebpack-dev-server是一个d开发时很好用的一个插件工具，当文件修改后保存时会在页面头部看到sever的状态变化，并且会进行热替换，实现页面的自动刷新


