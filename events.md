# 事件

- [基本用法](#basic-usage)
- [事件处理队列](#queued-event-handlers)
- [事件订阅者](#event-subscribers)
 
<a name="basic-usage"></a>
## 基本用法

Lumen 的事件功能简单的使用观察者模式实现, 允许你订阅或者监听应用里面发生的事件. 

#### 订阅事件

> **Note:** 如果你想使用 `Event` facade 的话, 请在 `bootstrap/app.php` 文件里面把 `$app->withFacades()`去掉注释. 

使用此方法 `Event::listen` 来订阅事件: 

	Event::listen(
		'PodcastWasPurchased', 'EmailPurchaseConfirmation@handle'
	);

> **Note:** 你可以把事件注册都写到 [服务提供者](/docs/providers) 文件里面.

当一个事件触发的时候, 事件对象会把调用监听者的  `handle`  方法: 

	class EmailPurchaseConfirmation {

		public function handle(PodcastWasPurchased $event)
		{
			//
		}

	}

你可以把监听者的类放在任何位置, 如统一放到 `app/Events`  文件夹里面.

#### 触发事件

现在我们准备好使用 `Event` facade 触发我们的事件：

	$response = Event::fire(new PodcastWasPurchased($podcast));

`fire` 方法返回一个响应的数组，让你可以用来控制你的应用程序接下来要有什么反应。

你也可以使用 `event` 辅助方法来触发事件:

	event(new PodcastWasPurchased($podcast));

#### 监听器闭包

你甚至可以不需对事件建立对应的处理类。举个例子，在你的 `EventServiceProvider` 的 `boot` 方法里，你可以做下面这件事：

	Event::listen('App\Events\PodcastWasPurchased', function($event) {
		// Handle the event...
	});

#### 停止继续传递事件

有时候你会希望停止继续传递事件到其他监听器。你可以通过从处理程序返回 `false` 来做到这件事：

	Event::listen('App\Events\PodcastWasPurchased', function($event) {
		// Handle the event...

		return false;
	});

<a name="queued-event-handlers"></a>
## 事件处理队列

需要把事件处理程序放到 [队列](/docs/queues) 里面吗？你可以使用 `Illuminate\Contracts\Queue\ShouldBeQueued` 接口来标识出来: 

	use Illuminate\Contracts\Queue\ShouldBeQueued;

	class SendPurchaseConfirmation implements ShouldBeQueued {

		public function (PurchasePodcast $event)
		{
			//
		}

	}

下次当某个事件被触发的时候, 监听器就会被自动加载到事件分发器里面去了. 

> **Note:**当然, 你需要配置 [队列设置](/docs/queues) 才能使用此功能.

当处理程序被队列执行，如果没有异常被丢出，在执行后该队列中的任务将会自动被删除。你也可以手动取用队列中的任务的 `delete` 和 `release` 方法。队列处理程序默认会引入的 `Illuminate\Queue\InteractsWithQueue` trait，让你可以取用这些方法: 

	public function handle(PodcastWasPurchased $event)
	{
		if (true) {
			$this->release(30);
		}
	}
