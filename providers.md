# 服务提供者

- [简介](#introduction)
- [基本提供者例子](#basic-provider-example)
- [注册提供者](#registering-providers)

<a name="introduction"></a>
## 简介

服务提供者是所有 Lumen 应用程序的启动中心。你的应用程序，以及所有 Lumen 的核心服务，都是透过服务提供者启动。

但我们所说的「启动」指的是什么？一般而言，我们指**注册**事物，包括注册服务容器绑定、事件监听器、过滤器，甚至路由。服务提供者是你的应用程序配置中心所在。

如果你打开包含于 Lumen 中的 `bootstrap/app.php` 这一文件，你会看到 `$app->register()` 的方法调用。你也可以通过调用这个方法来注册额外的服务提供者。

在这份概述中，你会学到如何编写你自己的服务提供者，并将它们注册于你的 Lumen 应用程序。

<a name="basic-provider-example"></a>
## 基本提供者例子

所有的服务提供者都应继承 `Illuminate\Support\ServiceProvider` 此一类。在这个抽象类中，至少必须定义一个方法： `register` 。

### 注册者方法

现在，让我们来看看基本的服务提供者：

	<?php namespace App\Providers;

	use Riak\Connection;
	use Illuminate\Support\ServiceProvider;

	class RiakServiceProvider extends ServiceProvider {

		/**
		 * 在容器中注册绑定。
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->singleton('Riak\Contracts\Connection', function($app) {
				return new Connection($app['config']['riak']);
			});
		}

	}

这个服务提供者只定义了一个 `register` 方法，并在服务容器中使用此方法定义了一份 `Riak\Contracts\Connection` 的实现。若你还不了解服务容器是如何运作的，不用担心，[我们很快会提到它](/docs/5.0/container)。

此类位于 `App\Providers` 命名空间之下，因为这是 Lumen 中默认服务提供者所在的位置。然而，你可以随自己的需要改变它。你的服务提供者可被置于任何 Composer 能自动加载的位置。


<a name="registering-providers"></a>
## 注册提供者

所有的服务提供者都在 `bootstrap/app.php` 文件里面注册, 此文件包含了一个简单的调用 `$app->register()`. 

如果你想注册自定义的服务提供者, 请使用: 

	$app->register('App\Providers\YourServiceProvider');
