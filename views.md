# 视图 (View)

- [基本用法](#basic-usage)

<a name="basic-usage"></a>
## 基本用法

视图里面包含了你应用程序所提供的 HTML 代码，并且提供一个简单的方式来分离控制器和网页呈现上的逻辑。视图被保存在 `resources/views` 文件夹内。

一个简单的视图看起来可能像这样：

	<!-- View stored in resources/views/greeting.php -->

	<!doctype html>
	<html>
		<head>
			<title>Welcome!</title>
		</head>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

这个视图可以使用以下的代码传递到用户的浏览器：

	$app->get('/', function() {
		return view('greeting', ['name' => 'James']);
	});

如你所见，`view` 辅助方法的第一个参数会对应到 `resources/views` 文件夹内视图文件的名称；传递到 `view` 辅助方法的第二个参数是一个能够在视图内取用的数据数组。

当然，视图文件也可以被存放在 `resources/views` 的子文件夹内。举例来说，如果你的视图文件保存在 `resources/views/admin/profile.php`，你可以用以下的代码来返回：

	return view('admin.profile', $data);

#### 传递数据到视图

	// 使用传统的方法
	$view = view('greeting')->with('name', 'Victoria');

	// 使用魔术方法
	$view = view('greeting')->withName('Victoria');

在上面的例子代码中，视图将可以使用 `$name` 来取得数据，其值为 `Victoria`。

如果你想的话，还有一种方式就是直接在 `view` 辅助方法的第二个参数直接传递一个数组：

	$view = view('greetings', $data);

如果你使用上面的方法来进行数据传参, `$data` 必须是 键/值 对应的数组数据, 这样在视图里面, 你可以使用对应的键来获取值, 如: `{{ $key }}` 会取得  `$data['$key']` 对应的数据. 

#### 确认视图是否存在

如果你需要确认视图是否存在，使用 `exists` 方法：

	if (view()->exists('emails.customer')) {
		//
	}

#### 从一个文件路径产生视图

你可以从一个完整的文件路径来产生一个视图：

	return view()->file($pathToFile, $data);
