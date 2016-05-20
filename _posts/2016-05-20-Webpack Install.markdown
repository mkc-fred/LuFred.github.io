---
layout: post
title: "WebPack初体验"
subtitle: "最近公司新的项目在前端换了一套新的一套View框架(React),所以原来那套Knockout＋requiseJS技术组合有点不适合用在这里，Google了一下发现一个用于React开发和模块管理的最佳工具是一个叫Webpack的货。所以下面就是对它的一个最基本了解。"
date: 2016-05-20
author: LuJiangBo
category: JavaScript
tags: JavaScript
finished: true
---

![webpack]({{ post.url| prepend: site.url  }}/content/images/201605/2016-05-20-webpackinstall01.png) 

## webpack是什么  
[Webpack](http://webpack.github.io)是当下热门的前端资源模块化管理和打包工具，它可以将松散的模块按照依赖规则打包成符合生产环节部署的前段资源。还可以将按需加载的模块进行代码分隔，等到实际需要的时候在异步加载，通过loader的转换，任何形式的资源都可以视作模块，比如CommonJs模块、AMD模块、CMD模块、ES6模块、CSS、image、JSON、LESS等。

## 安装   

### 1.安装Nodejs  
因为webpack在执行打包压缩的时候是依赖nodejs的，所以这里我们需要下载Nodejs。切后面所以包的下载都是需要通过nodejs的npm模块来下载。  
[下载地址](https://nodejs.org/en/)

### 2.初始化package.json
package.json中包含各种所需模块以及项目的配置信息(名称、版本、许可证等)，以及一些执行脚本也可以放在这。具体配置可以看[这里](https://docs.npmjs.com/files/package.json)  
命令:$ npm init   
`该命令会出创建一个名为package.json的文件，用来记录你项目中依赖的包`

### 3.安装webpack  
命令:$ npm install webpack -g  `-g参数表示webpack全局安装`

也可以在后面加--save-dev参数，表示只将包安装到当前项目  
&emsp;&emsp;$ npm install webpack --save-dev  
`--save-dev表示把webpack安装到当前目录的node_modules文件夹下`        

### 4.安装特定版本的webpack
你也可以安装特定版本的包文件  
&emsp;&emsp;$ npm install webpack@1.2.x --save-dev



## 用例
>下面是一个最简单的使用webpack的一个例子

### 1.创建一个cats.js 文件

{% highlight javascript %}
    var cats = ['dave', 'henry', 'martha'];
    module.exports = cats;
{% endhighlight %}

### 2.创建一个app.js 文件 依赖于cats.js

{% highlight javascript %}
    cats = require('./cats.js');
    console.log(cats);
{% endhighlight %}

### 3.打包js
你可以直接在命令行中打包资源，如打包app.js  
命令:webpack ./app.js app.bundle.js      
`成功后会创建生成一个打包后的app.bundle.js文件`

或者你也可以创建一个webpack.config.js文件，替代上面webpack命令后面的参数。  
手动创建一个名为webpack.config.js的文件。  
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

### 4.配置module节点  
webpack可以通过module加载不同功能的模块解析器，比如解析React的jsx文件  
首先得在项目目录安装相应的包，如下面的babel-loader
命令:npm i --save-dev babel-core babel-loader  
其次在config中配置module参数  
{% highlight javascript %}
        module: {
            loaders: [{                     //多个loader用!分隔 loader:'babel!jsx'
                test: /\.jsx?$/,
                exclude: /node_modules/,
                loader: 'babel',
            }]
        }
{% endhighlight %}  

### 5.插件依赖  
通常我们还需要额外的包来支持我们的项目，比如js的压缩，公共js代码的提取。
这时候我们可以在config文件中通过配置plugins来添加相应功能的插件包
如：
{% highlight javascript %}
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

## 番外  
$ npm install webpack-dev-server --save-dev  

>webpack-dev-server是一个d开发时很好用的一个插件工具，当文件修改后保存时会在页面头部看到sever的状态变化，并且会进行热替换，实现页面的自动刷新


