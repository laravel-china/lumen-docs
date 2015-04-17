# 会话

- [配置](#configuration)
- [使用 Session](#session-usage)
- [暂存数据（Flash Data）](#flash-data)
- [数据库 Sessions](#database-sessions)
- [Session 驱动](#session-drivers)

<a name="configuration"></a>
## 配置

由于 HTTP 协定是无状态（Stateless）的，所以 session 提供一种保存用户数据的方法。Lumen 和 Laravel 一样支持了多种 session 后端驱动，并通过清楚、统一的 API 提供使用。也内置支持如 [Memcached](http://memcached.org)、[Redis](http://redis.io) 和数据库的后端驱动。

在 `.env` 文件里的 `SESSION_DRIVER`  选项可以来控制会话的驱动器, 默认情况下 Lumen 使用 `memcached` 作为缓存驱动器. 

> **Note:** 请把 `bootstrap/app.php` 文件里面的 `Dotenv::load` 这个调用注释掉, 如果你想使用 `.env` 文件的话.

如果你想使用 redit 作为你的驱动器的话, 请使用 composer 安装 `predis/predis` 扩展包版本 (~1.0).

#### 保留键值

Lumen 框架在内部有使用 `flash` 作为 session 的键值，所以应该避免 session 使用此名称。

<a name="session-usage"></a>
## 使用

#### 开启会话功能

> **Note:** 你需要把 `bootstrap/app.php` 文件里面的 `$app->middleware()`去掉注释, 才能使用此功能.

#### 获取会话实例

有好几种方法可以获取到会话的实例, 一种是 HTTP 请求的 `session` 方法, 一种是 `Session` facade, 另一种是 `session` 辅助函数, 当 `session` 在被调用的时候没有传入任何参数, 会返回整个会话实例, 如下: 

	session()->regenerate();

#### 保存对象到 Session 中

	Session::put('key', 'value');

	session(['key' => 'value']);

#### 保存对象进 Session 数组值中

	Session::push('user.teams', 'developers');

#### 从 Session 取回对象

	$value = Session::get('key');

	$value = session('key');

#### 从 Session 取回对象，若无则返回默认值

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });

#### 从 Session 取回对象，并删除

	$value = Session::pull('key', 'default');

#### 从 Session 取出所有对象

	$data = Session::all();

#### 判断对象在 Session 中是否存在

	if (Session::has('users'))
	{
		//
	}

#### 从 Session 中移除对象

	Session::forget('key');

#### 清空所有 Session

	Session::flush();

#### 重新产生 Session ID

	Session::regenerate();

<a name="flash-data"></a>
## 暂存数据（Flash Data）

有时你可能希望暂存一些数据，并只在下次请求有效。你可以使用 `Session::flash` 方法来达成目的：

	Session::flash('key', 'value');

#### 刷新当前暂存数据，延长到下次请求

	Session::reflash();

#### 只刷新指定快闪数据

	Session::keep(['username', 'email']);

<a name="database-sessions"></a>
## 数据库 Sessions

当使用 `database` session 驱动时，你必需建置一张保存 session 的数据表。下方例子使用 `Schema` 来建表：

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

<a name="session-drivers"></a>
## Session 驱动

session 配置文件中的「driver」定义了 session Lumen 提供了许多良好的驱动：

- `file` - sessions 将保存在 `storage/framework/sessions`。
- `cookie` - sessions 将安全保存在加密的 cookies 中。
- `database` - sessions 将保存在你的应用程序数据库中。
- `memcached` / `redis` - sessions 将保存在一个高速缓存的系统中。
- `array` - sessions 将单纯的以 PHP 数组保存，只存活在当次请求。

> **注意：** array 驱动典型应用在 [unit tests](/docs/5.0/testing) 环境下，所以不会留下任何 session 数据。
