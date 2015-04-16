# Queues

- [配置信息](#configuration)
- [基本用法](#basic-usage)
- [更多调度的方法](#more-dispatch-methods)
- [队列闭包](#queueing-closures)
- [启动队列监听](#running-the-queue-listener)
- [常驻队列线程](#daemon-queue-worker)
- [失败的工作](#failed-jobs)

<a name="configuration"></a>
## 设置

Lumen 队列组件提供一个统一的 API 集成了许多不同的队列服务，队列允许你延后执行一个耗时的任务，例如延后至指定的时间才发送邮件，进而大幅的加快了应用程序处理请求的速度。

Lumen 支持 数据库, [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), [Redis](http://redis.io), null, 和 synchronous (本地开发使用) 队列驱动器. `null`队列驱动器会对所有的工作进行忽略, 并且从来不会运行. 

请在  `.env`  文件里面配置 `QUEUE_DRIVER` 参数来指定队列驱动器. 

### Q队列数据表

为了能够使用 `database` 驱动，你需要建立一个数据表来保存工作。要使用一个迁移建立这个数据表，下面是表结构.

	Schema::create('jobs', function(Blueprint $table)
	{
		$table->bigIncrements('id');
		$table->string('queue');
		$table->text('payload');
		$table->tinyInteger('attempts')->unsigned();
		$table->tinyInteger('reserved')->unsigned();
		$table->unsignedInteger('reserved_at')->nullable();
		$table->unsignedInteger('available_at');
		$table->unsignedInteger('created_at');
	});

### 其他队列依赖

下面的依赖是使用对应的队列驱动所需的扩展包：

- Amazon SQS: `aws/aws-sdk-php`
- Beanstalkd: `pda/pheanstalk ~3.0`
- IronMQ: `iron-io/iron_mq ~1.5`
- Redis: `predis/predis ~1.0`

<a name="basic-usage"></a>
## 基本用法

#### 推送一个工作至队列

应用程序中能够放进队列的工作都存放在 `App\Jobs` 目录下，所有的工作类都可以放到此文件夹下. 

> **Note:** 如果你想使用 `Queue` facade 的话, 请在 `bootstrap/app.php` 文件里面把 `$app->withFacades()`去掉注释. 

要推送一个新的工作至队列，请使用 `Queue::push` 方法：

	Queue::push(new SendEmail($message));

你也可以在路由闭包内, 或者控制器内使用 `dispatch` 方法: 

	$this->dispatch(new SendEmail($message));

命令里会包含一个 `handle` 方法, 这个方法将会在队列执行时被调用。你可以在 `handle` 方法使用时提示传入任何你需要的依赖，而 [服务容器](/docs/5.0/container) 会自动注入他们：

	public function handle(UserRepository $users)
	{
		//
	}

#### 指定队列使用特定连接

你也可指定队列工作送至指定的连接：

	Queue::pushOn('emails', new SendEmail($message));

#### 发送相同的数据去多个队列工作

如果你需要发送一样的数据去几个不同的队列工作，你可以使用 `Queue::bulk` 方法：

	Queue::bulk([new SendEmail($message), new AnotherCommand]);

#### 延迟执行一个工作

有时候你可能想要延迟执行一个队列工作，举例来说你希望一个队列工作在客户注册 15 分钟后才寄送 e-mail，你可以使用 `Queue::later` 方法来完成这件事情:

	$date = Carbon::now()->addMinutes(15);

	Queue::later($date, new SendEmail($message));

在这个例子中，我们使用 [Carbon](https://github.com/briannesbitt/Carbon) 日期类库来指定我们希望队列工作希望延迟的时间，另外你也可发送一个整数来设置你希望延迟的秒数。

> **注意:** 在 Amazon SQS 服务中，有最大 900 秒（ 15 分钟 ）的限制。

#### 将 Eloquent 模型放进队列

如果你队列工作的构造器接收一个 Eloquent 模型，只有这个模型的标记（ identifier ） 会被序列化后放到队列中。当工作真正开始被处理的时候，队列系统会自动从数据库中重新取得完整的模型实例。这个对你的网站应用程序来说是完全透明的，并且预防一些在序列化完整 Eloquent 模型实例时可能遇到的问题。


#### 删除一个处理中的工作

一旦一个工作被处理过后，这个工作必须从队列中删除。假如在工作执行后没有发生错误，这个将会自动完成。

如果你希望能够手动删除或着释放工作，在 `Illuminate\Queue\InteractsWithQueue` trait 中提供 `release` 以及 `delete` 方法的接口。其中 `release` 方法接受单一一个值：你想要等待工作再次能够执行的秒数。

	public function handle(SendEmail $job)
	{
		if (true) {
			$this->release(30);
		}
	}

#### 释放一个工作回到队列中

假如在工作执行后发生错误，这个工作将会自动被释放回到队列之中，如此一来便能够再次尝试执行工作。工作会一直被释放回队列直到到达应用程序的尝试上限。这个上限数值可以在使用 `queue:listen` 或 `queue:work` Artisan 命令时候借由 `--tries` 开关来设置。

#### 检查工作执行次数

当一个工作执行后发生错误，这个工作将会自动的释放回队列当中，你可以透过 `attempts` 方法来检查这个工作已经被执行的次数：

	if ($this->attempts() > 3) {
		//
	}

> **注意:** 你的命令处理类别必须使用 `Illuminate\Queue\InteractsWithQueue` 这个 trait 才能够使用这个方法。

<a name="more-dispatch-methods"></a>
## 更多调度的方法

### Mapping Command Properties From Requests

It is very common to map HTTP request variables into jobs. So, instead of forcing you to do this manually for each request, Lumen provides some helper methods to make it a cinch. Let's take a look at the `dispatchFrom` method available from Closure routes and controller methods:

	$this->dispatchFrom('Command\Class\Name', $request);

This method will examine the constructor of the job class it is given, and then extract variables from the HTTP request (or any other `ArrayAccess` object) to fill the needed constructor parameters of the job. So, if our job class accepts a `firstName` variable in its constructor, the job bus will attempt to pull the `firstName` parameter from the HTTP request.

You may also pass an array as the third argument to the `dispatchFrom` method. This array will be used to fill any constructor parameters that are not available on the request:

	$this->dispatchFrom('Command\Class\Name', $request, [
		'firstName' => 'Taylor',
	]);

<a name="queueing-closures"></a>
## Queueing Closures

> **Note:** 使用队列闭包的话, 你需要安装 `jeremeamia/superclosure` (~2.0) 

你也可以推送一个闭包去队列，这个方法非常的方便及快速的来处理需要使用队列的简单的任务：

#### 推送一个闭包至队列

	Queue::push(function($job) use ($id) {
		Account::delete($id);

		$job->delete();
	});

> **注意:** 要让一个组件变量可以在队列闭包中可以使用我们会通过 `use` 命令，试着发送主键及重复使用的相关模块在你的队列工作中，这可以避免其他的序列化行为。

当使用 Iron.io [push queues](#push-queues) 时,你应该在队列闭包中采取一些其他的预防措施，我们应该在执行工作收到队列数据时检查token是否真来自 Iron.io，举例来说你推送一个队列工作到 `https://yourapp.com/queue/receive?token=SecretToken`，接下来在你的工作收到队列的请求时，你就可以检查token的值是否正确。

<a name="running-the-queue-listener"></a>
## 执行一个队列监听

Lumen 内含一个 Artisan 命令，它将推送到队列的工作拉来下执行，你可以使用 `queue:listen` 命令，来执行这件常驻任务：

#### 开始队列监听

	php artisan queue:listen

你也可以指定特定队列连接让监听器使用：

	php artisan queue:listen connection

注意当这个任务开始时，这将会一直持续执行到他被手动停止，你也可以使用一个处理监控如 [Supervisor](http://supervisord.org/) 来确保这个队列监听不会停止执行。

你也可以在 `listen` 命令中使用逗号分隔的队列连接，来设置不同队列连接的优先层级：

    php artisan queue:listen --queue=high,low

在这个范列中，总是会优先处理 `high-connection` 中的工作，然后才处理 `low-connection`。

#### 指定工作超时参数

你也可以设置给每个工作允许执行的秒数：

    php artisan queue:listen --timeout=60

#### 指定队列休息时间

此外，你也可以指定让监听器在拉取新工作时要等待几秒：

	php artisan queue:listen --sleep=5

注意队列只会在工作时休息，假如有许多可执行的工作，队列会持续的处理工作而不会休息

#### 处理队列上的第一个工作

当你只想处理队列上的一个工作你可以使用 `queue:work` Artisan 命令：
	php artisan queue:work

<a name="daemon-queue-worker"></a>
## 常驻队列处理器

在 `queue:work` 中也包含了一个 `--daemon` 选项，强迫队列处理器持续处理工作，而不会每次都重新启动框架，这个作法比起 `queue:listen` 可有效减少 CPU 使用量，但是却增加了布署时，对于处理中队列任务的复杂性。

要启动一个常驻的队列处理器，使用 `--daemon`：

	php artisan queue:work connection --daemon

	php artisan queue:work connection --daemon --sleep=3

	php artisan queue:work connection --daemon --sleep=3 --tries=3

如你所见 `queue:work` 命令支持 `queue:listen` 大多相同的选项参数，你也可使用 `php artisan help queue:work` 命令来观看全部可用的选项参数。

### 布署常驻队列处理器

最简单布署一个应用程序使用常驻队列处理器的方式，就是将应用程序在开始布署时转成维护模式，你可以使用 `php artisan down` 命令来完成这件事情，当这个应用程序在维护模式，Lumen 将不会允许任何来自队列上的新工作，但会持续的处理已存在的工作。

要重新启动 `queue` 也是非常容易，请将底下命令加到部署命令：

	php artisan queue:restart

上述命令会在执行完目前的工作后，重新启动队列。

> **注意:** 这个命令依赖缓存系统来排定重新启动任务。默认 APCu 无法在命令提示字符中工作。如果你正在使用 APCu 请将 `apc.enable_cli=1` 加到你的 APCu 设置当中。

### 撰写常驻队列处理器

常驻队列处理器不会在处理每一个工作之前都重新启动框架。因此，你应该注意并小心地在工作处理完成之前释放占用的资源。例如，如果你正在使用 GD 函式库操作图片，当你完成工作的时候，你应该使用 `imagedestroy` 方法来释放占用的内存。

同样地，数据库连接可能在长时间执行的队列处理器中断线，你可以使用 `DB::reconnect` 方法来确保你每次都有一个全新的连接。

<a name="failed-jobs"></a>
## 已失败的工作

Since things don't always go as planned, sometimes your queued jobs will fail. Don't worry, it happens to the best of us! Lumen and Lumen include a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into a `failed_jobs` table.

The `failed_jobs` table should have a schema like the following:


事情往往不会如你预期的一样，有时候你推送工作到队列会失败，别担心，Lumen 包含一个简单的方法去指定一个工作最多可以被执行几次，在工作被执行到一定的次数时，他将会添加至 `failed_jobs` 数据表里，下面是表结构: 

	Schema::create('failed_jobs', function(Blueprint $table)
	{
		$table->increments('id');
		$table->text('connection');
		$table->text('queue');
		$table->text('payload');
		$table->timestamp('failed_at');
	});

你可以指定一个最大值来限制一个工作应该最多被执行几次，在你执行 `queue:listen` 时加上 `--tries`：

    php artisan queue:listen connection-name --tries=3

假如你会想注册一个事件，这个事件会将会在队列失败时被调用，你可以使用 `Queue::failing` 方法，这个事件是一个很好的机会让你可以通知你的团队通过 e-mail 或 [HipChat](https://www.hipchat.com)。

	Queue::failing(function($connection, $job, $data) {
		//
	});

你可能够直接在队列工作类别中定义一个 `failed` 方法，这让你能够在工作失败时候，执行一些特定的动作：

	public function failed()
	{
		// Called when the job is failing...
	}

### 重新尝试失败的工作

要看到所有失败的工作，你可以使用 `queue:failed` 命令：

    php artisan queue:failed

这个 `queue:failed` 命令将会列出工作 ID、连接、队列名称及失败的时间，可以使用工作 ID 重新执行一个失败的工作，例如一个已经失败的工作的 ID 是 5，我们可以使用下面的命令：

    php artisan queue:retry 5

假如你想删除一个失败的工作，可以使用 `queue:forget` 命令：

    php artisan queue:forget 5

要删除全部失败的工作，可以使用 `queue:flush` 命令：

    php artisan queue:flush
