# 服务容器

- [介绍](#introduction)
- [基本用法](#basic-usage)
- [绑定实例的接口](#binding-interfaces-to-implementations)
- [上下文绑定](#contextual-binding)
- [标签](#tagging)
- [容器事件](#container-events)

<a name="introduction"></a>
## 介绍

Lumen 服务容器是管理类依赖的强力工具。依赖注入是比较专业的说法，真正意思是将类依赖透过构造器或 「setter」 方法注入。

<a name="basic-usage"></a>
##  基本用法

> **Note:** 为了保证程序的整洁, 建议放置于 [服务提供者](/docs/providers) 里面.

#### 注册基本解析器

服务容器注册依赖有几种方式，包括闭包回调和绑定实例的接口。首先，我们来探讨闭包回调的方式。被注册至容器的闭包解析器包含一个 key (通常用类名称) 和一个有返回值的闭包:

	$app->bind('FooBar', function($app) {
		return new FooBar($app['SomethingElse']);
	});

#### 注册一个单例

有时候，你可能希望绑定到容器的对象只会被解析一次，之后的调用都返回相同的实例：

	$app->singleton('FooBar', function($app) {
		return new FooBar($app['SomethingElse']);
	});

#### 绑定一个已经存在的实例

你也可以使用 `instance` 方法，绑定一个已经存在的实例到容器，接下来将总是返回该实例：

	$fooBar = new FooBar(new SomethingElse);

	$app->instance('FooBar', $fooBar);

### 解析

从容器解析出实例有几种方式。  
一、可以使用 `make` 方法：

	$fooBar = $app->make('FooBar');

#### 自动解析

二、你可以在构造函数中简单地「类型指定（type-hint）」你所需要的依赖，包括在控制器、事件监听器、队列任务，过滤器等等之中。容器将自动注入你所需的所有依赖：

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;
	use App\Users\Repository as UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="binding-interfaces-to-implementations"></a>
## 将接口绑定到实现

服务容器有个非常强大特色，能够绑定特定实例的接口。举例，假设我们应用程序要集成 [Pusher](https://pusher.com) 服务去收发即时事件，如果使用 Pusher 的 PHP SDK，可以在类注入一个 Pusher 客户端实例. 

可以在服务容器像这样注册它：

	$app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

当有类需要 `EventPusher` 接口时，会告诉容器应该注入 `PusherEventPusher`，现在就可以在构造器中「类型指定」一个 `EventPusher` 接口：

		/**
		 * Create a new order handler instance.
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="contextual-binding"></a>
##  上下文绑定

有时候，你可能会有两个类需要用到同一个接口，但是你希望为每个类注入不同的接口实现。例如当我们的系统收到一个新的订单时，我们需要使用 [PubNub](http://www.pubnub.com/) 来代替 Pusher 发送消息。Lumen 提供了一个简单便利的接口来定义以上的行为：

	$app->when('App\Service\CreateOrder')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## 标签

偶尔你可能需要解析绑定中的某个「类」。例如你正在建设一个汇总报表，它需要接收实现了 `Report` 接口的不同实现的数组。在注册了 `Report` 的这些实现之后，你可以用 `tag` 方法来给他们赋予一个标签：

	$app->bind('SpeedReport', function() {
		//
	});

	$app->bind('MemoryReport', function() {
		//
	});

	$app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服务打上标签，可以通过 `tagged` 方法轻易地解析它们：

	$app->bind('ReportAggregator', function($app) {
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="container-events"></a>
## 容器事件

#### 注册一个解析事件监听器

容器在解析每一个对象时就会触发一个事件。你可以用 `resolving` 方法来监听此事件：

	$app->resolving(function($object, $app) {
		// Called when container resolves object of any type...
	});

	$app->resolving(function(FooBar $fooBar, $app) {
		// Called when container resolves objects of type "FooBar"...
	});

被解析的对象将被传入到闭包方法中。
