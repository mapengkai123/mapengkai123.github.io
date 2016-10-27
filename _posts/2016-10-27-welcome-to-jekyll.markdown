---
layout: post
title:  "使用jekyll搭建博客!"
date:   2016-10-27 11:10:38 +0800
categories: jekyll update
---
打造高端blog
前期准备：windows安装ruby和jekyll
先对环境的安装

Jekyll是一款静态网站生成工具，允许用户使用HTML、Markdown或Textile通过模块的方式建立所需网站，然后通过模板引擎Liquid（Liquid Templating Engine）来运行或者生成对应的静态网站文件. 
在GitHub上使用较多，通过GitHub搭建自己的博客一般来说就是使用Jekyll；因为GitHub的渲染引擎默认为Jekyll。

网上很多类似的安装教程，但是一般来说都是需要安装Python，在本篇文章中我们不使用“Pygments”代码高亮引擎，所以不需要安装Python。
安装 Ruby

Jekyll是一款基于Ruby的插件，安装Ruby是必须的. 
1. 下载，传送阵：http://rubyinstaller.org/downloads/ 
2. 点击版本并下载，这里我下载的是：“Ruby 2.2.1 (x64)” 
3. 点击进行安装，此时需要注意两点： 
*安装目录不允许包含空格 
*选中“Add Ruby executables to your PATH”这样将自动完成环境变量的配置。


4.完成后进入“CMD”输入“ruby -v”如显示版本则代表安装成功。
安装 DevKit

DevKit 是一个在 Windows 上帮助简化安装及使用 Ruby C/C++ 扩展如 RDiscount 和 RedCloth 的工具箱。
更多详细的安装指南请查看Ruby的 wiki 页面 阅读。

    前往 http://rubyinstaller.org/downloads/

    下载与 Ruby 版本相对应的 DevKit 安装包。 例如：“DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe” 
    版本对应关系：

        Ruby 1.8.7 and 1.9.3: 
        DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe

        2.0 and 2.1 (32bits version only): 
        DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe

        2.0 and 2.1 (x64 - 64bits only) 
        DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe

    运行文件选择解压目录，如“D:\ToolKits\Ruby\DevKit”

    解压完成后，通过初始化来创建 config.yml 文件。在命令行窗口内，输入下列命令：

        1 cd "D:\ToolKits\Ruby\DevKit"
        2 ruby dk.rb init
        3 notepad config.yml

    此时已经使用记事本打开所创建的”config.yml”文件，于末尾添加新的一行: “- D:\Ruby200-x64“，这里的目录为你的Ruby的安装目录，保存文件并退出。

    回到命令行窗口内进行安装。

        1 ruby dk.rb install

安装 Jekyll

    首先确保 gem 已经正确安装

        1  //命令输入
        2   gem -v
        3  //输出
        4   2.2.2

    安装 Jekyll
        1 //命令行执行
        2  gem install jekyll

    错误 
    在这里或许你将遇到一定的问题，比如：

    ERROR: Could not find a valid gem ‘jekyll’ (>= 0), here is why: 
    Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/latest_spece.4.8.gz)

这个错误的原因是因为证书问题，简单的解决办法为：下载最新的证书，放到指定文件夹，并配置环境变量。

下载：http://curl.haxx.se/ca/cacert.pem 
拷贝到：Ruby安装目录下的“bin”文件夹下 
环境变量： 
这里写图片描述

至于其他错误，比如443错误，这个多访问几次，或者挂上VPN进行，多尝试几次就OK。
安装 Rouge

一般来说静态生成中经常会使用高亮代码等功能，而高亮代码的生成一般需要插件帮助完成才行；在常规中一般都是使用：“Pygments”；因为”Pygments“是python下面的插件，所以需要先安装Python之后才能安装该插件，我嫌麻烦在实际使用中采用的是”Rouge“高亮插件。 
之所以使用：”Rouge”，是因为在 Jekyll 官网中也曾提到以后将会使用该插件。 
安装步骤非常简单，同样使用命令行安装就OK：

    1 //命令行 Gem 安装
    2 gem install rouge

一般来说有一定可能会遇到服务器没有响应或者 443 等错误，这些都无需担心，多尝试几次就OK。

这些前期准备都完成以后，我们就可以在本地建一个博客了，
     1 jekyll new yourblog
 输入命令后会在生成一个yourblog的文件夹，里面的文件大家可以自己研究，对后期开发有很大的帮助。
 接下来在命令行输入

     1 jekyll serve

 开启了服务后不可关闭，在浏览器访问localhost:4000便可以访问本地博客
 我们可以把本地的博客推到github 这样属于自己的博客就完成了！
[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
