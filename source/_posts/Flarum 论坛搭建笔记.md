---
title: Flarum 论坛搭建笔记
date: 2019-03-03 17:29:49
tags: 
    - flarum
    - 论坛
category:
    - 后台     
---
>  bbs.51tiki.com 就是使用flarum搭建的，再次感谢flarum开发社区

-   composer create-project flarum/flarum my_discuz dev-master
-   虚拟主机配置文件 include 项目目录下的 .nginx.conf 并进行相关修改;
-   chmod -R 775 ./my_discuz/
<!-- more -->
-   修改  
```bash
path: my_discuz/vendor/flarum/core/migrations/2015_02_24_000000_create_posts_table.php
```
-   注释 # $table->engine = 'MyISAM';
-   访问主页 填写相关配置 点击 install 

-   安装中文汉化包 composer require csineneo/lang-simplified-chinese
-   这个时候可以进入后台配置邮箱 （发件地址一定要填写自己的账号）
-   进入后台扩展包 选择 对勾 简体中文语言包
-   后台--basics-default language-选择简体中文即可（很关键 当时不知道在此处改，耽误了很久）
-   安装本地上传图片的扩展 composer require "flagrow/upload:*"
-   进入后台扩展包 选择 对勾 upload 然后进行配置
-   php flarum cache:clear（不能省略）
-   composer require s9e/flarum-ext-autovideo
-   composer require s9e/flarum-ext-autoimage:*
-   这两个是自动加载链接中的视频和图片(选装) --如果安装别忘了在后台勾选上此扩展（autoimage,autovideo）
-   composer require wiwatsrt/flarum-ext-best-answer 最佳回复
-   composer dump-autoload