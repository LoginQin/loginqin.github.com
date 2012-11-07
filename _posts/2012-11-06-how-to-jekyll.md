---
layout: default
title: Windows下安装jekyll本地测试环境
---
##{{page.title}}
  今天尝试着在本地搭建jekyll, 网上有不少文章, 但是在win7下还是遇到了不少问题, 这里补充记录一下关于jekyll本地GBK编码错误问题, 解决:  
1.windows环境变量加入
    
    LANG=en_US.UTF-8
    LC_ALL=en_US.UTF-8
    
  当然作为临时的, 在CMD下可以直接敲
    
    set LANG=en_US.UTF-8
    set LC_ALL=en_US.UTF-8

2._POST报错, 要保证文件名是英文的就没有问题, 上传到Github, 英文中文都可以.

###摘下网上参考的安装过程[云在千峰](http://yunfeng.sinaapp.com/?p=437#ixzz2BSIHktVB):  
1. 安装Ruby：在Windows系统上当然使用rubyinstaller了，[点击下载](http://rubyinstaller.org/downloads/) 
2. 安装Ruby DevKit[点击下载](https://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe)
3. 安装Jekyll
4. 安装Python[点击下载](http://portablepython.com/wiki/PortablePython3.2.1.1)
5. 安装Pygments

###详细步骤
1.从rubyinstaller下载安装包并安装到某个磁盘中，在安装界面把所有的选项都勾选上  

2.把下载的DevKit解压到某个目录 在该目录中运行如下命令：

    ruby dk.rb init
    ruby dk.rb install
    gem install jekyll

现在可以开始使用jekyll了。如果您还需要使用代码高亮工具，则需要继续安装Pygments ，过程如下：

3.安装下载的Portable Python（笔者使用的是PortablePython_3.2.1.1.exe），安装目录为E:\Portable_Python_3.2.1.1
然后把以下目录分别添加到系统Path环境变量中  

    E:\Portable_Python_3.2.1.1\App
    E:\Portable_Python_3.2.1.1\App\Scripts (注意如果刚安装py环境,可能没有这个Scripts路径, 可以自己提前建立, 当安装完某个工具,例如pygments, 才可能看见这个文件夹)
 

4.把下载的distribute-0.6.28.tar.gz解压的某个目录(比如：E:\distribute-0.6.28),猛击我下载  

在该目录中运行如下命令:  

    python distribute_setup.py

5.然后通过如下命令来安装pygments：  

    easy_install Pygments


然后就可以使用Jekyll了，在生成静态页面的时候 可能还会出现 GBK字符不能编码的问题，但是不影响生成网页了。







*{{page.date | date_to_string}}*
