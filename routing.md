# HTTP 路由

- [基础路由](#basic-routing)
- [路由参数](#route-parameters)
- [路由命名](#named-routes)
- [路由组](#route-groups)
- [CSRF 保护](#csrf-protection)
- [请求方法欺骗](#method-spoofing)
- [抛出 404 错误](#throwing-404-errors)

<a name="basic-routing"></a>
## 基础路由

您将在 `app/Http/routes.php` 中定义应用中的大多数路由，这个文件被 `bootstrap/app.php` 加载。 就如 Laravel 一样, 大部分路由只需要接受一个 URI 和 一个 闭包(Closure) 参数：

#### 基本的 GET 路由

	$app->get('/', function() {
		return 'Hello World';
	});

#### 其他基础路由

	$app->post('foo/bar', function() {
		return 'Hello World';
	});

	$app->patch('foo/bar', function() {
		//
	});

	$app->put('foo/bar', function() {
		//
	});

	$app->delete('foo/bar', function() {
		//
	});

你可以使用 `url` 辅助函数生成 URL : 

	$url = url('foo');

#### 把请求导向控制器

请查阅此文档 [controllers](/docs/controllers).

<a name="route-parameters"></a>
## 路由参数

你可以从 URL 中抓取你想要的参数。

#### 基本路由参数

	$app->get('user/{id}', function($id) {
		return 'User '.$id;
	});

#### 正则表达式约束

> **注意：** 这个地方和 Laravel 不兼容, 如果你想升级到 Laravel, 请把你的正则表达式约束放置在 `where` 方法里面。

	$app->get('user/{name:[A-Za-z]+}', function($name) {
		//
	});

<a name="named-routes"></a>
## 命名路由

命名路由让你更方便地产生 URL 与重定向特定路由。您可以用 `as` 的数组键值指定名称给路由：

	$app->get('user/profile', ['as' => 'profile', function() {
		//
	}]);

也可以为控制器动作指定路由名称：

	$app->get('user/profile', [
		'as' => 'profile', 'uses' => 'UserController@showProfile'
	]);

现在你可以使用路由名称产生 URL 或进行重定向：

	$url = route('profile');

	$redirect = redirect()->route('profile');

<a name="route-groups"></a>
## 路由群组

有时候, 你想对好几个路由进行统一逻辑处理, 如中间件, 你可以使用路由群组功能. 

<a name="route-group-middleware"></a>
### 中间件

中间件会对路由群组里面的路由进行作用, 使用参数 `middleware` 进行设定, 可以设定多个中间件, 请注意中间件的顺序是执行的顺序. 

	$app->group(['middleware' => 'foo|bar'], function($app)
	{
		$app->get('/', function() {
			// Uses Foo & Bar Middleware
		});

		$app->get('user/profile', function() {
			// Uses Foo & Bar Middleware
		});
	});

<a name="route-group-namespace"></a>
### 命名空间

您一样可以在 group 属性数组中使用 `namespace` 参数，指定在这群组中控制器的命名空间：

	$app->group(['namespace' => 'Admin'], function($app) {
		// Controllers Within The "App\Http\Controllers\Admin" Namespace
	});

<a name="csrf-protection"></a>
## CSRF 保护

> **Note:** 你必须 [开启会话](/docs/session#session-usage) 来使用此功能.

Lumen 提供简易的方法，让您可以保护您的应用程序不受到 CSRF ([跨网站请求伪造](http://en.wikipedia.org/wiki/Cross-site_request_forgery)) 攻击。跨网站请求伪造是一种恶意的攻击，借以代表经过身份验证的用户执行未经授权的命令。

Lumen 会自动在每一位用户的 session 中放置随机的 token ，这个 token 将被用来确保经过验证的用户是实际发出请求至应用程序的用户：

#### 插入 CSRF Token 到表单

	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

当然也可以在 Blade [模板引擎](/docs/templates) 使用：

	<input type="hidden" name="_token" value="{{ csrf_token() }}">

您不需要手动验证在 POST、PUT、DELETE 请求的 CSRF token。如果你在 `bootstrap/app.php` 文件里面开启的话, `Laravel\Lumen\Http\Middleware\VerifyCsrfToken` [HTTP 中间件](/docs/middleware) 将保存在 session 中的请求输入的 token 配对来验证 token 。

#### X-CSRF-TOKEN

除了寻找 CSRF token 作为「POST」参数，中间件也检查 `X-XSRF-TOKEN` 请求头，比如，你可以把 token 存放在 meta 标签中, 然后使用 jQuery 将它加入到所有的请求头中：

	<meta name="csrf-token" content="{{ csrf_token() }}" />

	$.ajaxSetup({
		headers: {
			'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
		}
	});

现在所有的 AJAX 请求会自动加入 CSRF token：

	$.ajax({
	   url: "/foo/bar",
	})

#### X-XSRF-TOKEN

Lumen 也在 cookie 中存放了名为 `XSRF-TOKEN` 的 CSRF token。你可以使用这个 cookie 值来设置 `X-XSRF-TOKEN` 请求头。一些 Javascript 框架，比如 Angular ，会自动设置这个值。

> Note:  `X-CSRF-TOKEN` 和 `X-XSRF-TOKEN` 的不同点在于前者使用的是纯文本而后者是一个加密的值，因为在 Lumen 中 cookies 始终是被加密过的。如果你使用 `csrf_token()` 函数来作为 token 的值， 你需要设置 `X-CSRF-TOKEN` 请求头。

<a name="method-spoofing"></a>
## 请求方法欺骗

HTML 表单没有支持 `PUT` 、`PATCH` 或 `DELETE` 请求。所以当定义 `PUT` 、`PATCH` 以及 `DELETE` 路由并在 HTML 表单中被调用的时候，您将需要添加隐藏 `_method` 字段在表单中。

发送的 `_method` 字段对应的值会被当做 HTTP 请求方法。举例来说：

	<form action="/foo/bar" method="POST">
		<input type="hidden" name="_method" value="PUT">
		<input type="hidden" name="_token" value="{{ csrf_token() }}">
	</form>

<a name="throwing-404-errors"></a>
## 抛出 404 错误

这里有两种方法从路由手动触发 404 错误。

首先，您可以使用 `abort` 辅助函数：

	abort(404);

`abort` 辅助函数只是简单抛出带有特定状态代码的 `Symfony\Component\HttpFoundation\Exception\HttpException` 异常 。

第二，您可以手动抛出 Symfony\Component\HttpKernel\Exception\NotFoundHttpException 的实体。

有关如何处理 404 异常状况和自定响应的更多信息，可以参考 [错误](/docs/errors#http-exceptions) 章节内的文档。

