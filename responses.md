# HTTP 响应

- [基本响应](#basic-responses)
- [重定向](#redirects)
- [其他响应](#other-responses)

<a name="basic-responses"></a>
## 基础响应

#### 从路由返回字串

最基本的响应就是从 Laravel 的路由返回字串：

	$app->get('/', function() {
		return 'Hello World';
	});

#### Creating Custom Responses

但是以大部分的路由及控制器所执行的动作来说，你需要返回完整的 `Illuminate\Http\Response` 实例或是一个[视图](/docs/views)。返回一个完整的 `Response` 实例时，你能够自定义响应的 HTTP 状态码以及响应头。`Response` 实例继承了 `Symfony\Component\HttpFoundation\Response` 类，它提供了很多方法来建立 HTTP 响应。

	use Illuminate\Http\Response;

	return (new Response($content, $status))
	              ->header('Content-Type', $value);

为了方便起见，你可以使用辅助方法 `response`：

	return response($content, $status)
	              ->header('Content-Type', $value);

> **提示：** 有关 `Response` 方法的完整列表可以参照 [API 文档](http://laravel.com/api/5.0/Illuminate/Http/Response.html) 以及 [Symfony API 文档](http://api.symfony.com/2.5/Symfony/Component/HttpFoundation/Response.html).

<a name="redirects"></a>
## 重定向

重定向响应通常是类 `Illuminate\Http\RedirectResponse` 的实例，并且包含用户要重定向至另一个 URL 所需的响应头。

#### 返回重定向

有几种方法可以产生 `RedirectResponse` 的实例，最简单的方式就是透过辅助方法 `redirect`。当在测试时，建立一个模拟重定向响应的测试并不常见，所以使用辅助方法通常是可行的：

	return redirect('user/login');

#### 返回重定向并且加上快闪数据（ Flash Data ）

> **Note:** 你需要开启 [会话](/docs/session#session-usage) 来使用功能.

通常重定向至新的 URL 时会一并将[数据存进一次性 Session](/docs/5.0/session)。所以为了方便，你可以利用方法连接的方式创建一个 `RedirectResponse` 的实例**并**将数据存进一次性 Session：

	return redirect('user/login')->with('message', 'Login Failed');

#### 返回根据前一个 URL 的重定向

你可能希望将用户重定向至前一个位置，例如当表单提交之后。你可以使用 `back` 方法来达成这个目的：

	return redirect()->back();

	return redirect()->back()->withInput();

#### 返回根据路由名称的重定向

当你调用辅助方法 `redirect` 且不带任何参数时，将会返回 `Illuminate\Routing\Redirector` 的实例，你可以对该实例调用任何的方法。举个例子，要产生一个 `RedirectResponse` 到一个路由名称，你可以使用 `route` 方法：

	return redirect()->route('login');

#### 返回根据路由名称的重定向，并给予路由参数赋值

如果你的路由有参数，你可以放进 `route` 方法的第二个参数。

	// For a route with the following URI: profile/{id}

	return redirect()->route('profile', ['id' => 1]);

如果你要重定向至路由且路由的参数为 Eloquent 模型的「ID」，你可以直接将模型传入，ID 将会自动被提取：

	return redirect()->route('profile', ['id' => $user]);

#### 返回根据路由名称的重定向，并给予特定名称路由参数赋值

	// For a route with the following URI: profile/{user}

	return redirect()->route('profile', ['user' => 1]);

<a name="other-responses"></a>
## 其他响应

使用辅助方法 `response` 可以轻松的产生其他类型的响应实例。

#### 建立 JSON 响应

`json` 方法会自动将响应头的 `Content-Type` 配置为 `application/json`：

	return response()->json(['name' => 'Abigail', 'state' => 'CA']);

#### 建立 JSONP 响应

	return response()->json(['name' => 'Abigail', 'state' => 'CA'])
	                 ->setCallback($request->input('callback'));

#### 建立文件下载的响应

	return response()->download($pathToFile);

	return response()->download($pathToFile, $name, $headers);

	return response()->download($pathToFile)->deleteFileAfterSend(true);

> **提醒：**管理文件下载的扩展包，Symfony HttpFoundation，要求下载文件名必须为 ASCII。
