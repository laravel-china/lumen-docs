
# 缓存

- [配置](#configuration)
- [基本使用](#basic-usage)

<a name="configuration"></a>
## 配置

在 `.env` 文件中, 有个  `CACHE_DRIVER`  的选项, 用来配置使用哪个类型的缓存, Lumen 支持以下的几种: 

- `array`
- `file`
- `memcached`
- `redis`
- `database`

> **Note:** 如果你需要使用 `.env` 来管理你的配置信息的话, 请在 `bootstrap/app.php` 文件里面把这一行去掉注释 `Dotenv::load()`.

### Memcached

如果你想使用 Memcached 缓存的话, 请在 `.env` 文件里面设置这两个选项  `MEMCACHED_HOST` 和 `MEMCACHED_PORT` .

### Redis

如果你想使用 Redis 缓存的话, 你需要通过 composer 安装 `predis/predis` package (~1.0) 扩展包. 

### Database

如果你打算用数据库作为缓存的话, 你需要配置好数据库表后才能使用, 下面是表结构: 

	Schema::create('cache', function($table) {
	    $table->string('key')->unique();
	    $table->text('value');
	    $table->integer('expiration');
	});

<a name="basic-usage"></a>
## 基础使用

> **Note:** 如果你想使用 `Cache` 的话, 请在 `bootstrap/app.php` 文件中把 `$app->withFacades()` 这一行去掉注释.

#### 保存

	Cache::put('key', 'value', $minutes);

#### 使用 Carbon 来设置过期时间

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### 如果不存在的话再保存

	Cache::add('key', 'value', $minutes);

此 `add` 会返回是否加入 Cache 的反馈, 如果增加成功的话, 会返回 `true`, 否则 `false`. 

#### 检查是否存在

	if (Cache::has('key')) {
		//
	}

#### 读取

	$value = Cache::get('key');

#### 读取如果不存在的话返回默认值

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

#### 永久性存储一份数据

	Cache::forever('key', 'value');

有时候您会希望从缓存中取得对象，而当此对象不存在时会保存一个默认值，您可以使用 `Cache::remember` 方法：

	$value = Cache::remember('users', $minutes, function() {
		return DB::table('users')->get();
	});

您也可以结合 `remember` 和 `forever` 方法：

	$value = Cache::rememberForever('users', function() {
		return DB::table('users')->get();
	});

请注意所有保存在缓存中的对象皆会被序列化，所以您可以任意保存各种类型的数据。

#### 从缓存拉出对象

如果您需要从缓存中取得对象后将它删除，您可以使用 `pull` 方法：

	$value = Cache::pull('key');

#### 从缓存中删除对象

	Cache::forget('key');
