---
title: windows 使用docker-compose+DockerFile部署项目(内含laravel-echo-server)
date: 2020-09-15 14:03:28
category:
    - 服务端   
tags: 
    - docker
    - laravel-echo-server
    - supervisor
thumbnail: /thumbnails/lv1.png
---
> 平时开发环境是windows下的集成环境，在一年前下载了Docker Desktop 想安装laradock 但是忘了什么原因了 失败了，最近闲来没事准备整一下本地的dockfile一键部署

**本文的阅读和实践需要一些docker、lnmp、docker命令等基础。**
**重申一下 这是windows环境下laravel项目的部署。**
>感谢laradock项目给我的很多地方的灵感，让我自己编写dockerfile时得心应手。大家在部署的时候也可以参考它的源代码
## 目录
1. [Docker Desktop 的安装 以及 配置](#d1)
2. [docker-compose.yml + dockerfile 文件说明](#d2)
3. [laravel-echo-server的部署指南](#d3)
4. [supervisor 监控队列](#d4)
5. [容器内的定时任务](#d5)
6. [k8s 入门调试(下一篇文章吧)](#d5)
 <!-- more -->
#### <a href="#d1">1.Docker Desktop 的安装 以及配置</a> 
就是官网下载安装，修改镜像源(Setting-Docker Engine-registry-mirrors,可以去DAOCloud找个源)。现在的desktop 更新的很智能了，不像之前还要手动添加允许file sharing，现在都随着挂载随着提示你允许，并且应该是自动安装好docker-compose,如果没有安装，就百度下安装方式吧。
#### <a href="#d2">2.docker-compose.yml + dockerfile 文件说明</a> 
**先看下大体的项目目录吧**
# project
-  app
-  config
- docker
- - app_local.dockerfile（workspace容器部署文件）
- - supervisor.conf（supervisord 配置文件）
- - crontabs（容器内的定时任务）
- - laravel-echo-server.dockerfile
- - package.json(单独安装laravel-echo-server 的)
- - php.ini
- - vhost.conf
- - web.dockerfile
- public
- resources
- ....
- .env
- .laravel-echo-server.json
- .docker-compose.yml
- .....
- - -
- 项目目录下创建docker-compose.yml和 docker/ *.dockfile,然后根据项目具体需求调整文件内容，下面我把用过的所有服务都列到下面举个例子。直接在代码里说明吧
**composer和node我没有选择使用镜像容器处理，因为可扩展性不是很好，平时更新项目时可能需要修改composer和npm，我也下载过镜像进行模拟 使用起来都不尽如人意，不如直接在php app的容器(workspace)中进行命令操作。**
```shell
version: '3.7'
services:
  app:
    image: xxx_app #镜像名称
    build: #指定该容器构建参数
      context: ./
      dockerfile: ./docker/app_local.dockerfile #dockerfile 的目录在当前项目下的docker文件下
    working_dir: /app # 指定容器工作目录
    restart: always
    volumes: #目录挂载 修改php配置
      - ./:/app
      - ./docker/php.ini:/usr/local/etc/php/conf.d/php.ini #配置文件
    environment: #这里可以添加修改.env的参数使其使用容器服务 
      - REDIS_HOST=redis

  # nginx
  web:
    image: xxxx_web
    build:
      context: ./
      dockerfile: ./docker/web.dockerfile
    restart: always
    depends_on:
      - app
    ports:
      - 80:80  #前面的80指本地端口 后面的80指容器中的端口
    volumes: #本机也需要记录日志
      - ./:/app
      - E:/DockerData/nginx/:/data/logs/nginx/

  # redis database 需要连接本地数据库时可以使用
  redis:
    container_name: xxx_redis #不使用dockfile的时候设置容器名称
    volumes:
      - E:/DockerData/redis:/data  # 挂载redis数据目录
      - E:/DockerData/conf/redis/redis.conf:/etc/redis/redis.conf  # 需要配置文件 不然无法启动
    image: redis
    command: redis-server /etc/redis/redis.conf
    ports:
      - 6380:6379

# mysql database 需要连接本地数据库时可以使用 
  database:
    container_name: xxx_mysql
    image: mysql:5.7
    command: --default-authentication-plugin=mysql_native_password
    ports:
    - 3316:3306
    environment: #MYSQL_ROOT_PASSWORD 才是登陆密码 env中的密码
    - "MYSQL_ROOT_PASSWORD=password"
    - "MYSQL_DATABASE=database_name"
    volumes: #需要数据转移的时候 可以挂载 没有的话就直接进入php容器 执行数据迁移
    - D:/xxxx/MySQL5.7.26/data/xxx:/var/lib/mysql/xxxx
  
  #  laravel-echo-server
  laravel-echo-server:
    build:
      context: ./
      dockerfile: ./docker/laravel-echo-server.dockerfile
    volumes:
      - ./laravel-echo-server.json:/usr/src/app/laravel-echo-server.json
    ports:
      - 6001:6001 #关键 别设置错
    links:
      - redis #通知关联redis服务
```
- 下面我们来看dockfile和配置文件
1. app_local.dockerfile (workspace 也就是php-fpm)

``` shell
FROM php:7.3-fpm-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.huaweicloud.com/g' /etc/apk/repositories
RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
        freetype \
        libpng \
        libjpeg-turbo \
        freetype-dev \
        libpng-dev \
        jpeg-dev \
        libjpeg \
        libjpeg-turbo-dev \
        libwebp-dev\
        libzip-dev\
        git
RUN docker-php-ext-configure gd \
                --with-freetype-dir=/usr/lib/ \
                --with-png-dir=/usr/lib/ \
                --with-jpeg-dir=/usr/lib/ \
                --with-webp-dir=/usr/lib/
RUN docker-php-ext-install mysqli pdo pdo_mysql gd zip
WORKDIR /app
RUN apk add composer npm
RUN composer config -g repos.packagist composer https://packagist.phpcomposer.com
RUN npm config set registry https://mirrors.huaweicloud.com/repository/npm/

# composer 和 npm的命令等容器创建好之后 进入容器进行 
# composer install composer autoload-dump 和 npm install npm run prod 等操作
```
2. laravel-echo-server.dockfile(关于laravel-echo-server 使用和安装中的坑 可以看我之前的文章[laravel-echo-server 踩坑记录](https://learnku.com/articles/48310 "laravel-echo-server 踩坑记录"))

```shell
FROM node:alpine

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# 拷贝 属于 echo-server自己的package.json 与项目本身的区分开
COPY ./docker/package.json /usr/src/app/

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

RUN apk add --update \
    python \
    python-dev \
    py-pip \
    build-base

RUN npm install

# 根据项目自身的配置laravel-echo-server.json进行启动
COPY laravel-echo-server.json /usr/src/app/laravel-echo-server.json

EXPOSE 3000
CMD [ "npm", "start", "--force" ] #容器构建后 启动echo-server
```
3. package.json （laravel-echo-server 使用的）

```shell
{
  "name": "laravel-echo-server-docker",
  "description": "Docker container for running laravel-echo-server",
  "version": "0.0.1",
  "license": "MIT",
  "dependencies": {
    "laravel-echo-server": "^1.5.0"
  },
  "scripts": {
    "start": "laravel-echo-server start"
  }
}
```
4. php.ini(在yml文件中挂载了)

```shell
post_max_size = 500M
upload_max_filesize = 500M
memory_limit = 512M
#...其他配置的更改
```

5. vhost.conf（nginx 配置 更具体的内容可以看看laradock项目）

```shell

client_max_body_size 500m;


proxy_read_timeout 240s;

server {
    listen 80;

    index index.php index.html;
    root /app/public;

    server_name xxxx.com;

    access_log /data/logs/nginx/xxxx_access.log;
    error_log /data/logs/nginx/xxxx_error.log;

    location / {
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000; #workspace的服务名称app
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```
6. web.dockerfile （nginx的镜像文件）

```shell
FROM nginx:1.17-alpine
ADD docker/vhost.conf /etc/nginx/conf.d/default.conf
RUN mkdir -p /data/logs/nginx/
#这里修改日志名称 与上面的文件相对应
RUN touch /data/logs/nginx/xxxx_access.log
#这里修改与上面的文件相对应
RUN touch /data/logs/nginx/xxx_error.log
```
## <a href="#d3">laravel-echo-server的部署指南</a>
7. **laravel-echo-server.json（authhost不修改的话会显示 Error authenticating）**
```json
{
	"authHost": "http://web",
	"authEndpoint": "/broadcasting/auth",
	"clients": [],
	"database": "redis",
	"databaseConfig": {
		"redis": { //这里要修改的
			"port": "6379",
			"host": "redis"
		}
	},
	"devMode": true,
	"host": null,
	"port": "6001",
	"protocol": "http",
	"socketio": {},
	"sslCertPath": "",
	"sslKeyPath": ""
}
```

8. 最后提示一下.env文件别忘了修改 QUEUE_CONNECTION、BROADCAST_DRIVER为redis，DB_HOST修改为服务名称，我这里叫database。以及涉及到的主机挂载目录是否都已经创建？
***
然后你就可以愉快的 docker-compose up -d ,docker-compose build 了

#### <a href="#d4">supervisor 监控队列</a>
**先呈上supervisor.conf**

```shell
[program:yourname_queue]#这里改个自己名字
process_name=%(program_name)s_%(process_num)02d
directory=/app #你的容器内的项目目录 
command=php artisan queue:work --tries=3 --sleep=3 --daemon
autostart=true 
autorestart=true
numprocs=1
user=root
stopasgroup=true
killasgroup=true
redirect_stderr=true
stderr_logfile=/app/storage/logs/err_queue.log #配置错误日志文件路径
stdout_logfile=/app/storage/logs/queue.log  #配置输出日志文件路径
[supervisord]

[supervisorctl]

```
1. 在你的workspace容器内安装supervisor（直接配置dockfile即可）, 然后copy配置文件到容器内

```shell
# 我这里是app_local.dockerfile
RUN apk add supervisor
# supervisor 服务配置文件
COPY ./docker/supervisor.conf /etc/supervisor/conf.d/supervisord.conf
```
2. 在镜像完成成为容器后执行启动命令

```shell
docker exec your_container_name /usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
```

#### <a href="#d5">容器内的定时任务</a>
**呈上crontabs文件**

```shell
#最好不要用windows记事本编辑这个文件出现换行，以免出现不必要的bug
* * * * * php /app/artisan schedule:run >> /app/storage/logs/crond.log 2>&1
```
> 因为我的workspace镜像使用的是alpine系统，不是centos和ubuntu之类的，所以它自带的crontab与其他的不同，这里我就列出alpine的步骤，其他系统的方式应该是先安装 然后需要修改crontab文件的权限为600 然后执行。

1.在dockerfile中copy我们自己的crontab文件(可选)，也可以直接在命令行中echo方式书写一个crontab文件

```shell
# 可选
COPY ./docker/crontabs /etc/cron.d/
# 添加定时任务在docker中，似乎只能运行root的crontab命令，其路径在/var/spool/cron/crontabs/root中，并不是扫描cron.d/文件夹下面的所有文件，将自己的crontab脚本先copy到docker，再cat到docker的root脚本后面才可以运行
RUN cat /etc/cron.d/crontabs>>/var/spool/cron/crontabs/root

# 如果是不copy文件 直接书写crontab的方式（可选）
RUN echo -e '* * * * * php /app/artisan schedule:run >> /app/storage/logs/crond.log 2>&1'>>/var/spool/cron/crontabs/root
```

2. 在镜像完成成为容器后执行启动命令（类似supervisor），如果不这样做的话 直接在dockerfile中CMD或者Entrypoint的话，我尝试过会报错。

```shell
docker exec your_container_name crond
```
> 这里我参考的文章链接[在alpine linux构建的docker中使用crontab执行定时任务](https://blog.csdn.net/bigheadsnake/article/details/78392539 "在alpine linux构建的docker中使用crontab执行定时任务")
里面也包含了修改容器 date 时区的方式 我就不赘述了。

**再重申一遍 我这是alpine的镜像 其他linux的镜像方式可百度**

***
好啦 结束了。以后若有新的dockerfile新增，我也会更新的。