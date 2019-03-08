---
title: 原生安装lnmp环境后的命令
date: 2019-02-27 17:29:46
tags:
    - nginx
    - lnmp
category:
    - 服务端 
thumbnail: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1552035234380&di=8c34c2b4602124ea3b396569f630125b&imgtype=0&src=http%3A%2F%2Fdn-linuxcn.qbox.me%2Fdata%2Fattachment%2Falbum%2F201611%2F22%2F214235y7ee099gwe97ieiy.png  
---
*因为之前使用的Linux系统都是安装的lnmp一键安装包或者docker下安装，所以对于原生安装lnmp环境的常用命令不是很熟悉，特此记录一下*


```bash
//nginx 相关
ps -ef| grep nginx  //查看配置文件路径
./nginx -t -c /usr/local/src/nginx/conf/nginx.conf
./nginx -t
./nginx -s reload 


//php-fpm 相关
/etc/php.ini
/etc/init.d/php-fpm start 
service php-fpm restart

//打开防火墙 80端口
iptables -I INPUT -p tcp --dport 80 -j ACCEPT


```