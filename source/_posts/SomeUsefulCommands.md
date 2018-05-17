---

title: Some Useful Commands
date: 2018-05-15 19:36:35
tags: commands
categories: 学习知识

---

## init

#### hexo init [folder]

新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

<!-- more -->

## new page

#### hexo new page [folder]
新建一个界面,在source文件夹下，将生成一个文件夹.

例如: hexo new page "about"

会在source文件夹下，将生成about文件夹,部署后将在public文件夹生成一个新的html页面：hexo\public\about\index.html,
通过访问https://用户名.github.io/about/访问这个页面,通过编辑index.html，添加自己的简历等其他你想加入的内容。但注意，
它是新建了一个html页面，不是文章，不会出现在博文目录。


## new

#### hexo new [layout] <title> ####

新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout(即 post) 参数代替。如果标题包含空格的话，请使用引号括起来。

## server ##

#### hexo server ####

启动服务器。默认情况下，访问网址为： http://localhost:4000/。

## clean ##

#### hexo clean

清除缓存文件 (db.json) 和已生成的静态文件 (public)。

## generate ##

#### hexo generate(hexo g) ####

生成静态文件。

## deploy ##

#### hexo deploy(hexo d)

部署网站。

hexo clean  (清理)

### 一般的在上传代码的时候需要三部 , 先clean, 在generate ,最后deploy部署上去 ###
### 本地调试一般使用 server 启动服务器来进行调试 ###

