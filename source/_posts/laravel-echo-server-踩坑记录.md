---
title: laravel-echo-server 踩坑记录
date: 2020-09-15 14:03:19
tags: laravel-echo-server
---
> 除了https://learnku.com/laravel/t/13101/using-laravel-echo-server-to-build-real-time-applications 文章中的记录以外 （下面也有包含的， 如果再次出现证明就是我自己曾经犯的错误）

- 需要取消注释 /config/app.php providers中的
```php
 App\Providers\BroadcastServiceProvider::class,//如果注释了就没有办法进行私有频道的开发了
```
- /config/database.php 中redis 的前缀注释掉
```php
    'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
//            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
```
- laravel-echo-server.json 中的配置
```
	"devMode": false,//取消开发模式 除非你想看到具体的链接情况
```
- 每次修改完的测试 都要进行对应重启：事件等后台业务修改 重启队列；前端等修改需要重新打包
```
laravel-echo-server start
npm run dev
php artisan queue:work
```
- 出现 Client can not be authenticated, got HTTP status xxx 类似情况 那就是laravel后台的认证问题，建议检查/routes/channel.php 文件中的频道 以及 guard， 修改完后 重启

> 若是在docker中部署laravel-echo-server,可以看我的另外一篇文章