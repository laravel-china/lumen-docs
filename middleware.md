# HTTP 中间件

- [简介](#introduction)
- [建立中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
- [可终止中间件](#terminable-middleware)

<a name="introduction"></a>

## 简介

HTTP 中间件提供一个方便的机制来过滤进入应用程序的 HTTP 请求，例如，Lumen 默认包含了一个中间件来检验用户身份验证，如果用户没有经过身份验证，中间件会将用户导向登录页面，然而，如果用户通过身份验证，中间件将会允许这个请求进一步继续前进。

当然，除了身份验证之外，中间件也可以被用来执行各式各样的任务，CORS 中间件负责替所有即将离开程序的响应加入适当的响应头，一个日志中间件可以记录所有传入应用程序的请求。
Lumen 框架已经内置一些中间件，包括维护、身份验证、CSRF 保护，等等。所有的中间件都位于 `app/Http/Middleware`  目录内。


<a name="defining-middleware"></a>
## 建立中间件

要建立一个新的中间件，只需要创建一个类, 并在类里面创建方法: 

	public function handle($request, $next)
	{
		return $next($request);
	}

举个例子, 在这个中间件内我们只允许 `年龄` 大于 200 的才能访问路由，否则，我们会将用户重新导向 「home」 的 URI 。

	<?php namespace App\Http\Middleware;

	class OldMiddleware {

		/**
		 * Run the request filter.
		 *
		 * @param  \Illuminate\Http\Request  $request
		 * @param  \Closure  $next
		 * @return mixed
		 */
		public function handle($request, Closure $next)
		{
			if ($request->input('age') < 200) {
				return redirect('home');
			}

			return $next($request);
		}

	}

如你所见，若是 `年龄` 小于 `200` ，中间件将会返回 HTTP 重定向给客户端，否则，请求将会进一步传递到应用程序。只需调用带有 `$request` 的 `$next` 方法，即可将请求传递到更深层的应用程序(允许跳过中间件)。

HTTP 请求在实际碰触到应用程序之前，最好是可以层层通过许多中间件，每一层都可以对请求进行检查，甚至是完全拒绝请求。

### *Before* / *After* 中间件

请求前后执行顺序, 指定某个中间件取决于这个中间件自身。这个中间件可以执行在请求前执行一些 **前置** 操作, 注意 `BeforeMiddleware`：

	<?php namespace App\Http\Middleware;

	class BeforeMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			// Perform action

			return $next($request);
		}
	}

然后，这个中间件也可以在请求后执行一些 **后置** 操作, 注意 `AfterMiddleware`：


	<?php namespace App\Http\Middleware;

	class AfterMiddleware implements Middleware {

		public function handle($request, Closure $next)
		{
			$response = $next($request);

			// Perform action

			return $response;
		}
	}

<a name="registering-middleware"></a>
## 注册中间件

### 全局中间件

若是希望中间件被所有的 HTTP 请求给执行，只要将中间件的类加入到 `bootstrap/app.php` 的 `$app->middleware()` 调用里面.

### 指派中间件给路由

如果你想为指定的路由加上中间件, 你需要先在 `bootstrap/app.php` 设置键值, 默认情况下, `$app->routeMiddleware()`  方法定义了应用的中间件, 想添加自定义的中间件, 只需要做如下: 

    $app->routeMiddleware([
        'old' => 'App\Http\Middleware\OldMiddleware',
    ]);

一旦你设定了中间件, 你可以使用 `middleware` 键在路由的选项参数里面, 见下面: 

	$app->get('admin/profile', ['middleware' => 'old', function() {
		//
	}]);

<a name="terminable-middleware"></a>
## 可终止中间件

有些时候中间件需要在 HTTP 响应已被发送到用户端之后才执行，例如，Laravel 内置的 「session」 中间件，保存 session 数据是在响应已被发送到用户端 _之后_ 才执行。为了做到这一点，你需要定义中间件为「可终止的」。


	use Illuminate\Contracts\Routing\TerminableMiddleware;

	class StartSession implements TerminableMiddleware {

		public function handle($request, $next)
		{
			return $next($request);
		}

		public function terminate($request, $response)
		{
			// Store the session data...
		}

	}

如你所见，除了定义 `handle` 方法之外， `TerminableMiddleware` 定义一个 `terminate`  方法。这个方法接收请求和响应。一旦定义了 terminable 中间件，你需要将它增加到 HTTP kernel 文件的全局中间件清单列表中。
