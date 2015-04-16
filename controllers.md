# HTTP 控制器

- [介绍](#introduction)
- [基础控制器](#basic-controllers)
- [控制器中间件](#controller-middleware)
- [依赖注入和控制器](#dependency-injection-and-controllers)

<a name="introduction"></a>
## 介绍

除了在单一的 `routes.php` 文件中定义所有的请求处理逻辑之外，你可能希望使用控制器类来组织此行为。控制器可将相关的 HTTP 请求处理逻辑组成一个类。控制器通常存放在 `app/Http/Controllers` 此目录中。

<a name="basic-controllers"></a>
## 基础控制器

这里是一个基础控制器类的例子: 

	<?php namespace App\Http\Controllers;

	use App\User;
	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			return view('user.profile', ['user' => User::findOrFail($id)]);
		}

	}

我们可以通过如下方式引导路由至对应的控制器动作：

	$app->get('user/{id}', 'App\Http\Controllers\UserController@showProfile');

> **Note:**  所有的控制器都应该扩展 `App\Http\Controllers\Controller` 基础控制器类.

#### 控制器和命名空间

和闭包路由一样，你也可以指定控制器路由的名称。

	$app->get('foo', ['uses' => 'App\Http\Controllers\FooController@method', 'as' => 'name']);

命名路由可以用以下方法来生成 URL : 

	$url = route('name');

带参数的命名路由: 

	$url = route('name', ['id' => 1]);

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/middleware) 可在控制器路由中指定，例如：

	$app->get('profile', [
		'middleware' => 'auth',
		'uses' => 'App\Http\Controllers\UserController@showProfile'
	]);

你也可以在控制器构造器中指定中间件 ：

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}

	}

<a name="dependency-injection-and-controllers"></a>
## 依赖注入和控制器

#### 构造器注入

Lumen / Laravel [服务容器](/docs/container) 用于解析所有的 Laravel 控制器。因此，你可以在控制器所需要的构造器中，对依赖作任何的类型限制。

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;
	use App\Repositories\UserRepository;

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

	}

#### 方法注入

除了建构器注入外，你也可以对控制器方法的依赖作类型限制。例如，让我们对某个方法的 `Request` 实例作类型限制：

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * 保存用户.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}

	}

如果你的控制器方法预期由路由参数取得输入，只要在其他的依赖之后列出路由参数即可：

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * 保存用户.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}
