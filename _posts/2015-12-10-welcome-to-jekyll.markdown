---
layout: post
title:  "使用jekyll搭建github blog"
date:   2015-12-11 22:14:54
categories: Jekyll
excerpt: 
---

* content
{:toc}


## 序

Github的博客打架还是足足耗了大半天，这里还是记个备忘。

---

### 步骤一 注册github账号

首先先要在github上注册账号了，这步应该没有什么问题。

### 步骤二 安装git客户端

肯定是要在本地安装git，我们一般都用windows的PC：
[https://git-for-windows.github.io/](https://git-for-windows.github.io/)

### 步骤三 创建github工程

登录github上，创建工程，这里有几点需要注意：

1. Github有两种pages模式。第一种是User/Organization Pages，用于个人或公司站点 ，注意我们要搭建的Blog需要使用这个模式。	
	- 使用自己的用户名，每个用户名下面只能建立一个；
	- 资源命名必须符合这样的规则username/username.github.com；
	- 主干上内容被用来构建和发布页面
2. Project Pages 项目站点	
	- gh-pages分支用于构建和发布；
	- 如果user/org pages使用了独立域名，那么托管在账户下的所有project pages将使用相同的域名进行重定向，除非project pages使用了自己的独立域名；
	- 如果没有使用独立域名，project pages将通过子路径的形式提供服务username.github.com/projectname；
	- 自定义404页面只能在独立域名下使用，否则会使用User Pages 404；
3. 创建项目站点步骤，这里注意项目名称必须为<username>.github.io：

    ![ruby-gems]({{ "/css/pics/20151220/github.io.repository.name.png"}})

### 步骤四 连接github

上传项目，注意如果不是首次提交只需要执行3、4、6：

1. 首先在本地文件创建于第三步同名的目录<username>.github.io 
2. 执行git init命令初始化目录
3. git add . 将当前目录的改动缓存本地
4. git commit -m "first post" 提交到本地
5. 将远程仓库在本地添加一个引用：origin  
    `git remote add origin https://github.com/<username>/<username>.github.io.git`
6. 将改动提交到github上 git push origin master

### 步骤五 准备jekyll环境

准备Jekyll调试环境，Github page有多种框架，本文介绍的是Jekyll。这步很关键，我们需要在本地搭建一个Jekyll的调试环境，这样才能所见即所得的调试我们编辑的网页，否则，我们每次修改都需要提交到github上才能看到结果，这个方式效率太低了，整个blog的过程和用到工具，见网上下面总结的图，这部是整个流程最麻烦的过程，大家要耐心，这部过去了基本就是通途了。

   ![ruby-gems]({{ "/css/pics/20151220/github.step.png"}})

首先我们需要现在本地安装一个ruby运行时环境，因为jekyll是基于ruby开发的。

   a)需要安装一个RubyInstaller
   
   b)然后安装DevKit，注意和RubyInstaller版本匹配

    cd C:\DevKit
    ruby dk.rb init
    ruby dk.rb install

   c)然后是安装Jekyll ：gem install Jekyll

   d)校验安装是否成功：jekyll --version

   e) 安装Rdiscount，这个用来解析Markdown标记的包，使用如下命令：


    gem install rdiscount

   f) OK,我们现在可以开启cmd启动jekyll了， 后面注意在我们工程目录下执行此命令：

    jekyll serve --safe --watch

 ![ruby-gems]({{ "/css/pics/20151220/start.jekyll.png"}})


这之后我们需要大概了解下jekyll的工程的目录结构，推荐参考：  
[http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html "http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html")  
当然也可以找个模板后，具体来看哈。

### 步骤六 制作blog模板并上传

OK，万事具备，我们去找个模板。
去这个网址，找个自己顺眼的现在本地调下：  
[http://jekyllthemes.org/](http://jekyllthemes.org/ "http://jekyllthemes.org/")  
有些在启动jekyll可能会报错，是因为缺少依赖的组件导致，自己根据错误信息度娘找下。

后面的事情大概就是修改模板，熟悉markdown如何用，建议下个好点的编辑器，我用的markdown就很不错，编辑blog基本就是几步：  
1. 先用markdownpad编辑一个md文件，放到模板的_posts目录下，基本就是用例子改下，分钟上手的事情哈  
2. 图片处理稍麻烦，有条件搞个图床，我就土点，每次都是先把图片放到模板的css目录下，然后在markdown中用图片标签引用  
3. 然后就在本地启动jekyll调试页面了，一般是http://localhost:4000，不同模板可能不同，见命令行中启动jekyll的输出了   
4. 调试无问题了，就用第四步骤中的3、4、6提交到github上面  
5. OK，可以用<username>.github.io访问了！

至此流程打通了，后面就看自己的了，呵呵，有精力还可以换换模板，完善下界面。

参考：  
[http://www.pchou.info/web-build/2013/01/03/build-github-blog-page-01.html](http://www.pchou.info/web-build/2013/01/03/build-github-blog-page-01.html "http://www.pchou.info/web-build/2013/01/03/build-github-blog-page-01.html")  
[http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html "http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html")

