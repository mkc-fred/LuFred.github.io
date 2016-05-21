---
layout: post
title: "DotNet Core安装篇"
subtitle: "在'跨平台'、'开源'，这些关键字大行其道的今天，我们的大微软不甘落后终于也推出了.Net Core，它是参考.Net Framework开发的一套跨平台.Net基础。既然他说跨平台那今天咱是不是就直接在Mac环境下敲一个Hellow World出来。下面就是在Mac环境下搭建的 .Net Core环境。"
date: 2016-05-21
author: LuJiangBo
category: DotNet Core
tags: DotNetCore
finished: true
---


## .Net Core是什么  
在[维基百科](https://zh.wikipedia.org/wiki/.NET_Core)上它是这样介绍的:  
.NET Core 是.NET Framework的新一代版本，是微软开发的第一个官方版本，具有跨平台 (Windows、Mac OSX、Linux) 能力的应用程序开发框架 (Application Framework)，未来也将会支持 FreeBSD 与 Alpine 平台，也是微软在一开始发展时就开源的软件平台，它经常也会拿来和现有的开源 .NET 平台 Mono 比较。  

由于 .NET Core 的开发目标是跨平台的 .NET 平台，因此 .NET Core 会包含 .NET Framework 的类的库，但与 .NET Framework 不同的是 .NET Core 采用包化 (Packages) 的管理方式，应用程序只需要获取需要的组件即可，与 .NET Framework 大包式安装的作法截然不同，同时各包亦有独立的版本线 (Version line)，不再硬性要求应用程序跟随主线版本。

## 安装  

### 1.准备工作  

在使用.NET Core前, 你需要通过[brew](http://brew.sh)安装OpenSSL,然后再执行下面的命令:   

>brew update  
>brew install openssl  
>brew link --force openssl

### 2.安装.NET Core SDK
在开始安装前，为确保成功，你需要删除你先前安装的.Net Core版本，通过下面的命令:  
直接复制下面代码贴到Terminal中执行即可。  
{% highlight Windows batch files %}
#!/usr/bin/env bash
#
# Copyright (c) .NET Foundation and contributors. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for full license information.
#

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

current_user=$(whoami)
if [ $current_user != "root" ]; then
    echo "$(basename "$0") uninstallation script requires superuser privileges to run"
    exit 1
fi

# this is the common suffix for all the dotnet pkgs
dotnet_pkg_name_suffix="com.microsoft.dotnet"
dotnet_install_root="/usr/local/share/dotnet"
dotnet_path_file="/etc/paths.d/dotnet"

remove_dotnet_pkgs(){
    installed_pkgs=($(pkgutil --pkgs | grep $dotnet_pkg_name_suffix))
    
    for i in "${installed_pkgs[@]}"
    do
        echo "Removing dotnet component - \"$i\""
        pkgutil --force --forget "$i"
    done
}

remove_dotnet_pkgs
[ "$?" -ne 0 ] && echo "Failed to remove dotnet packages." && exit 1

echo "Deleting install root - $dotnet_install_root"
rm -r "$dotnet_install_root"
rm "$dotnet_path_file"

echo "dotnet packages removal succeeded."
exit 0

{% endhighlight %}
然后再[下载官方PKG包](https://go.microsoft.com/fwlink/?LinkID=798400)。  

### 3.初始化代码 
通过下面的3条命令，我们就可以初始化一个最简单的Hello World 应用了。  

>mkdir hwapp  
>cd hwapp  
>dotnet new


### 4.运行应用      
下面2条命令分别是初始化project.json文件中依赖的包，第二条命令就是允许Hellow World了。
>dotnet restore  
>dotnet run  

![结果]({{ post.url| prepend: site.url  }}/content/images/201605/2016-05-21-DotNetCore01.png)  

### 5.完成  
到这里，我们在MAC环境下的.Net Core算是搭建完成了。官方原文以及其他系统下面的安装可以👀[这里](https://www.microsoft.com/net/core#macosx)。


