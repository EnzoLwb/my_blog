---
title: Laravel配合Supervisor实现秒级定时任务
date: 2019-02-27 17:30:04
tags:
    - Laravel
category:
    - 后台

---
> 一个进程需要每时每刻不断的跑，但是这个进程又有可能由于各种原因有可能中断。当进程中断的时候我希望能自动重新启动它，此时，我就需要使用到了Supervisor.
- supervisor 安装 
- 编写 laravel artisan command
<!-- more -->
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class Supervisor extends Command
{
	/**
	 * The name and signature of the console command.
	 *
	 * @var string
	 */
	protected $signature = 'supervisor:test {seconds}';

	/**
	 * The console command description.
	 *
	 * @var string
	 */
	protected $description = '测试 supervisor 秒级定时任务';

	/**
	 * Create a new command instance.
	 *
	 * @return void
	 */
	public function __construct()
	{
		parent::__construct();
	}

	/**
	 * Execute the console command.
	 *
	 * @return mixed
	 */
	public function handle()
	{
	    //记录日志
        Log::warning('time:现在时间 '.time());
	    //定时输出时间戳
        $seconds = $this->argument('seconds');
        usleep($seconds * 1000);
	}
}

```

- 修改supervisor 配置文件 并重载

```bash
[program:task-worker]
command=/usr/local/php/bin/php /home/www/wwwroot/hhwx/artisan supervisor:test 2
autostart=true
autorestart=true
user=root
numprocs=1
redirect_stderr=true
stdout_logfile=/home/www/wwwroot/hhwx/storage/logs/worker.log
```
- 查看日志

```
/usr/bin/supervisord -c /etc/supervisord.conf
supervisorctl stop all
ps aux | grep supervisord
```