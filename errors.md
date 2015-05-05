
# 错误和记录

- [配置](#configuration)
- [错误处理](#handling-errors)
- [HTTP 异常](#http-exceptions)
- [记录](#logging)

<a name="configuration"></a>
## 配置

Lumen 使用 [Monolog](https://github.com/seldaek/monolog), 一个兼容 [PSR-3](http://www.php-fig.org/psr/psr-3/) 标准的日志记录器. 

默认情况下, 日志记录器把所有的日志记录到一个文件里面, 并放置于 `storage/logs` 文件夹里, 然而因为 Lumen 使用了功能齐全的 [Monolog](https://github.com/Seldaek/monolog) , 可以随时配置成你想要的记录需求.

### 错误详情

在 `.env` 里面的选项 `APP_DEBUG` 可以控制错误详情在浏览器的输出.

> **Note:** 在的开发环境中, 建议把 `APP_DEBUG` 选项设置为 `true`, 在生成环境下, 请必须设置为 `false`.

<a name="handling-errors"></a>
## 错误处理

所有的异常都由 `App\Exceptions\Handler` 类处理。这个类包含两个方法： `report` 和 `render` 。

`report` 方法用来记录异常或把异常传递到外部服务，例如： [BugSnag](https://bugsnag.com) 。默认情况下， `report`  方法只基本实现简单地传递异常到父类并于父类记录异常。然而，你可以依你所需自由地记录异常。如果你需要使用不同的方法来报告不同类型的异常，你可以使用 PHP 的 `instanceof` 比较运算符：


	/**
	 * Report or log an exception.
	 *
	 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
	 *
	 * @param  \Exception  $e
	 * @return void
	 */
	public function report(Exception $e)
	{
		if ($e instanceof CustomException) {
			//
		}

		return parent::report($e);
	}

`render` 方法负责把异常转换成应该被传递回浏览器的 HTTP 响应。默认情况下，异常会被传递到基础类并帮你产生响应。然而，你可以自由的检查异常类型或返回自定义的响应。

异常处理进程的 `dontReport` 属性是个数组，包含应该不要被纪录的异常类型。由 404 错误导致的异常默认不会被写到日志文件。你可以依照需求添加其他类型的异常到这个数组。

<a name="http-exceptions"></a>
## HTTP 异常

有一些异常是描述来自服务器的 HTTP 错误码。例如，这可能是个「找不到页面」错误 (404)、「未授权错误」(401)，或甚至是工程师导致的 500 错误。使用下面的方法来返回这样一个响应：

	abort(404);

或是你可以选择提供一个响应：

	abort(403, 'Unauthorized action.');

你可以在请求的生命周期中任何时间点使用这个方法。

<a name="logging"></a>
## 日志

> **Note:** 如果你想使用 `Log` facade, 请把 `bootstrap/app.php` 文件里面的 `$app->withFacades()` 注释去掉.

Laravel 日志工具在强大的 [Monolog](http://github.com/seldaek/monolog) 函数库上提供一层简单的功能。Laravel 默认为应用程序建立每天的日志文件在 `storage/logs` 目录。你可以像这样把信息写到日志：

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');

日志工具提供定义在 [RFC 5424](http://tools.ietf.org/html/rfc5424)  的七个级别：**debug**、**info**、**notice**、**warning**、**error**、**critical** 和 **alert**。

也可以传入上下文相关的数据数组到日志方法里：

	Log::info('Log message', ['context' => 'Other helpful information']);

#### 从容器里面拿出 Logger 对象

可以使用以下方法拿到 logger 对象:

	$logger = app('Psr\Log\LoggerInterface');

当然, 你也可以利用 Lumen 的自动依赖注入的功能, 在路由闭包和控制器类里面使用. 
