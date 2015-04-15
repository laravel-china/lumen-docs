# 表单验证

- [基本用法](#basic-usage)
- [路由 / 控制器验证](#controller-validation)
- [使用错误信息](#working-with-error-messages)
- [错误信息 & 视图](#error-messages-and-views)
- [可用验证规则](#available-validation-rules)
- [添加条件验证规则](#conditionally-adding-rules)
- [自定义错误信息](#custom-error-messages)
- [自定义验证规则](#custom-validation-rules)

<a name="basic-usage"></a>
## 基本用法

Lumen 和 Laravel 一样，通过 `Validation` facade 让您可以简单、方便的验证数据正确性及查看相应的验证错误信息。

#### 基本验证例子

	$validator = Validator::make(
		['name' => 'Dayle'],
		['name' => 'required|min:5']
	);

上文中传递给 `make` 这个方法的第一个参数用来设定所需要被验证的数据名称，第二个参数设定该数据可被接受的规则。

#### 使用数组来定义规则

多个验证规则可以使用"|"符号分隔，或是单一数组作为单独的元素分隔。

	$validator = Validator::make(
		['name' => 'Dayle'],
		['name' => ['required', 'min:5']]
	);

#### 验证多个字段

	$validator = Validator::make(
		[
			'name' => 'Dayle',
			'password' => 'lamepassword',
			'email' => 'email@example.com'
		],
		[
			'name' => 'required',
			'password' => 'required|min:8',
			'email' => 'required|email|unique:users'
		]
	);

当一个 `Validator` 实例被建立后，`fails`（或 `passes`） 这两个方法就可以在验证时使用，如下：

	if ($validator->fails())
	{
		// The given data did not pass validation
	}

假如验证失败，您可以从验证器中接收错误信息。

	$messages = $validator->messages();

您可能不需要错误信息，只想取得无法通过验证的规则，您可以使用 `failed` 方法：

	$failed = $validator->failed();

#### 验证文件

`Validator` 类提供了一些规则用来验证文件，例如 `size`, `mimes` 等等。当需要验证文件时，您仅需将它们和您其他的数据一同送给验证器即可。

### 验证后钩子

验证器也允许你在完成验证后增加回调函数。这也允许你可以进行更进一步的验证，甚至在消息集合中增加更多的错误信息。我们在验证器实例中使用 `after` 方法来作为开始：

	$validator = Validator::make(...);

	$validator->after(function($validator) {
		if ($this->somethingElseIsInvalid()) {
			$validator->errors()->add('field', 'Something is wrong with this field!');
		}
	});

	if ($validator->fails()) {
		//
	}

您可以根据需要为验证器增加任意的 `after` 回调函数。

<a name="controller-validation"></a>
## 路由 / 控制器验证

当然，如果每一次需要验证的时候都手动的建立并且验证 `Validator` 实例会非常的麻烦。不用担心，你有其他的选择！Lumen 自带的 `Laravel\Lumen\Routing\Controller` 基类使用了一个 `ValidatesRequests` 的 trait。这个 trait 提供了一个单一的、便捷的方法来验证 HTTP 请求。代码如下：

	/**
	 * Store the incoming blog post.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function store(Request $request)
	{
		$this->validate($request, [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		]);

		//
	}

你甚至可以从一个路由闭包中调用 `validate` 方法：

	use Illuminate\Http\Request;

	$app->post('comment', function(Request $request) {

		$this->validate($request, [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		]);

		//
	});

如果验证通过了，你的代码会正常继续执行。如果验证失败，那么会抛出一个 `Illuminate\Contracts\Validation\ValidationException` 异常。这个异常会被自动捕获，然后重定向至用户上一个页面。而错误信息甚至已经存储至 session 中！

如果收到的是一个 AJAX 请求，那么不会生成一个重定向。相反的，一个带有 422 状态码的 HTTP 响应会被返回给浏览器，包含了一个含有错误信息的 JSON 对象。

比如，如下是手动创建验证的等效写法：

	/**
	 * Store the incoming blog post.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function store(Request $request)
	{
		$v = Validator::make($request->all(), [
			'title' => 'required|unique|max:255',
			'body' => 'required',
		]);

		if ($v->fails()) {
			return redirect()->back()->withErrors($v->errors());
		}

		//
	}

### 自定义闪存后的错误格式

如果你想要自定义验证失败后已经闪存至 session 的错误消息格式，可以通过覆盖基类控制器的 `formatValidationErrors`。不要忘记在文件顶部引入 `Illuminate\Validation\Validator` 类：

	/**
	 * {@inheritdoc}
	 */
	protected function formatValidationErrors(Validator $validator)
	{
		return $validator->errors()->all();
	}

如果你想要自定义通过路由闭包使用的 `validate` 方法时抛出的验证错误，可以调用 `Laravel\Lumen\Routing\Closure` 类：

	use Laravel\Lumen\Routing\Closure;

	Closure::formatErrorsUsing(function($validator) {
		return $validator->errors()->all();
	});

同样的, 你也可以在路由闭包的验证错误信息被渲染时自定义整个的 HTTP 响应：

	use Laravel\Lumen\Routing\Closure;

	Closure::buildResponseUsing(function($validator, $errors) {
		// Return Illuminate\Http\Response Instance...
	});

<a name="working-with-error-messages"></a>
## 使用错误信息

当您调用一个 `Validator` 实例的 `messages` 方法后，您会得到一个命名为 `MessageBag` 的实例，该实例里有许多方便的方法能让您取得相关的错误信息。

#### 查看一个字段的第一个错误信息

	echo $messages->first('email');

#### 查看一个字段的所有错误信息

	foreach ($messages->get('email') as $message) {
		//
	}

#### 查看所有字段的所有错误信息

	foreach ($messages->all() as $message) {
		//
	}

#### 判断一个字段是否有错误信息

	if ($messages->has('email')) {
		//
	}

#### 错误信息格式化输出

	echo $messages->first('email', '<p>:message</p>');

#### 查看所有错误信息并以格式化输出

	foreach ($messages->all('<li>:message</li>') as $message) {
		//
	}

<a name="error-messages-and-views"></a>
## 错误信息 & 视图

> **注意：** 在开始使用 Lumen 的这个特色前，你需要 [开启 sessions](/docs/session#session-usage).

当您开始进行验证数据时，您会需要一个简易的方法去取得错误信息并返回到您的视图中，在 Lumen 中您可以很方便的处理这些操作，您可以通过下面的路由例子来了解：

	$app->get('register', function() {
		return view('user.register');
	});

	$app->post('register', function() {
		$rules = [...];

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails()) {
			return redirect('register')->withErrors($validator);
		}
	});

需要记住的是，当验证失败后，我们会使用 `withErrors` 方法来将 `Validator` 实例进行重定向。这个方法会将错误信息存入 session 中，这样才能在下个请求中被使用。

然而，我们并不需要特别去将错误信息绑定在我们 GET 路由的视图中。因为 Laravel 会确认在 Session 数据中检查是否有错误信息，并且自动将它们绑定至视图中。**所以请注意，`$errors` 变量存在于所有的视图中，所有的请求里**，让您可以直接假设 `$errors` 变量已被定义且可以安全地使用。`$errors` 变量是 `MessageBag` 类的一个实例。

所以，在重定向之后，您可以自然的在视图中使用 `$errors` 变量：

	<?php echo $errors->first('email'); ?>

### 命名错误清单

假如您在一个页面中有许多的表单，您可能希望为错误命名一个 `MessageBag`。 这样能方便您针对特定的表单查看其错误信息, 我们只要简单的在 `withErrors` 的第二个参数设定名称即可：

	return redirect('register')->withErrors($validator, 'login');

接着您可以从一个 `$errors` 变量中取得已命名的 `MessageBag` 实例：

	<?php echo $errors->login->first('email'); ?>

<a name="available-validation-rules"></a>
## 可用验证规则

以下是现有可用的验证规则清单与他们的函数名称：

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Boolean](#rule-boolean)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [String](#rule-string)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

字段值为 _yes_, _on_, 或是 _1_ 时，验证才会通过。这在确认"服务条款"是否同意时很有用。

<a name="rule-active-url"></a>
#### active_url

字段值通过 PHP 函数 `checkdnsrr` 来验证是否为一个有效的网址。

<a name="rule-after"></a>
#### after:_date_

验证字段是否是在指定日期之后。这个日期将会使用 PHP `strtotime` 函数验证。

<a name="rule-alpha"></a>
#### alpha

字段仅全数为字母字串时通过验证。

<a name="rule-alpha-dash"></a>
#### alpha_dash

字段值仅允许字母、数字、破折号（-）以及底线（_）

<a name="rule-alpha-num"></a>
#### alpha_num

字段值仅允许字母、数字

<a name="rule-array"></a>
#### array

字段值仅允许为数组

<a name="rule-before"></a>
#### before:_date_

验证字段是否是在指定日期之前。这个日期将会使用 PHP `strtotime` 函数验证。

<a name="rule-between"></a>
#### between:_min_,_max_

字段值需介于指定的 _min_ 和 _max_ 值之间。字串、数值或是文件都是用同样的方式来进行验证。


<a name="rule-confirmed"></a>
#### confirmed

字段值需与对应的字段值 `foo_confirmation` 相同。例如，如果验证的字段是 `password` ，那对应的字段 `password_confirmation` 就必须存在且与 `password` 字段相符。

<a name="rule-date"></a>
#### date

字段值通过 PHP `strtotime` 函数验证是否为一个合法的日期。

<a name="rule-date-format"></a>
#### date_format:_format_

字段值通过 PHP `date_parse_from_format` 函数验证符合 _format_ 制定格式的日期是否为合法日期。

<a name="rule-different"></a>
#### different:_field_

字段值需与指定的字段 _field_ 值不同。


<a name="rule-digits"></a>
#### digits:_value_

字段值需为数字且长度需为 _value_。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

字段值需为数字，且长度需介于 _min_ 与 _max_ 之间。

<a name="rule-boolean"></a>
#### boolean

字段必须可以转换成布尔值，可接受的值为 `true`, `false`, `1`, `0`, `"1"`, `"0"`。

<a name="rule-email"></a>
#### email

字段值需符合 email 格式。

<a name="rule-exists"></a>
#### exists:_table_,_column_

字段值需与存在于数据库 _table_ 中的 _column_ 字段值其一相同。

#### Exists 规则的基本使用方法

	'state' => 'exists:states'

#### 指定一个自定义的字段名称

	'state' => 'exists:states,abbreviation'

您可以指定更多条件且那些条件将会被新增至 "where" 查询里：

	'email' => 'exists:staff,email,account_id,1'
	/* 这个验证规则为 email 需存在于 staff 这个数据库表中 email 字段中且 account_id=1 */

通过`NULL`搭配"where"的缩写写法去检查数据库的是否为`NULL`

	'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image

文件必需为图片(jpeg, png, bmp, gif 或 svg)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

字段值需符合事先给予的清单的其中一个值

<a name="rule-integer"></a>
#### integer

字段值需为一个整数值

<a name="rule-ip"></a>
#### ip

字段值需符合 IP 位址格式。

<a name="rule-max"></a>
#### max:_value_

字段值需小于等于 _value_。字串、数字和文件则是判断 `size` 大小。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

文件的 MIME 类需在给定清单中的列表中才能通过验证。

#### MIME规则基本用法

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

字段值需大于等于 _value_。字串、数字和文件则是判断 `size` 大小。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

字段值不得为给定清单中其一。

<a name="rule-numeric"></a>
#### numeric

字段值需为数字。

<a name="rule-regex"></a>
#### regex:_pattern_

字段值需符合给定的正规表示式。

**注意:** 当使用`regex`模式时，您必须使用数组来取代"|"作为分隔，尤其是当正规表示式中含有"|"字串。

<a name="rule-required"></a>
#### required

字段值为必填。

<a name="rule-required-if"></a>
#### required\_if:_field_,_value_

字段值在 _field_ 字段值为 _value_ 时为必填。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

字段值 _仅在_ 任一指定字段有值情况下为必填。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

字段值 _仅在_ 所有指定字段皆有值情况下为必填。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

字段值 _仅在_ 任一指定字段没有值情况下为必填。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

字段值 _仅在_ 所有指定字段皆没有值情况下为必填。

<a name="rule-same"></a>
#### same:_field_

字段值需与指定字段 _field_ 等值。

<a name="rule-size"></a>
#### size:_value_

字段值的尺寸需符合给定 _value_ 值。对于字串来说，_value_ 为需符合的字串长度。对于数字来说，_value_ 为需符合的整数值。对于文件来说，_value_ 为需符合的文件大小（单位 kb)。

<a name="rule-timezone"></a>
#### timezone

字段值通过 PHP `timezone_identifiers_list` 函数来验证是否为有效的时区。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

字段值在给定的数据库中需为唯一值。如果 `column（字段）` 选项没有指定，将会使用字段名称。

#### 唯一(Unique)规则的基本用法

	'email' => 'unique:users'

#### 指定一个自定义的字段名称

	'email' => 'unique:users,email_address'

#### 强制唯一规则忽略指定的 ID

	'email' => 'unique:users,email_address,10'

#### 增加额外的 Where 条件

您也可以指定更多的条件式到 "where" 查询语句中：

	'email' => 'unique:users,email_address,NULL,id,account_id,1'

上述规则为只有 `account_id` 为 `1` 的数据列会做唯一规则的验证。

<a name="rule-url"></a>
#### url

字段值需符合 URL 的格式。

> **注意:** 此函数会使用 PHP `filter_var` 方法验证。

<a name="conditionally-adding-rules"></a>
## 添加条件验证规则

某些情况下，您可能 **只想** 当字段有值时，才进行验证。这时只要增加 `sometimes` 条件进条件列表中，就可以快速达成：

	$v = Validator::make($data, [
		'email' => 'sometimes|required|email',
	]);

在上述例子中，`email` 字段只会在当其在 `$data` 数组中有值的情况下才会被验证。

#### 复杂的条件式验证

有时，您可以希望给指定字段在其他字段长度有超过 100 时才验证是否为必填。或者您希望有两个字段，当其中一字段有值时，另一字段将会有一个默认值。增加这样的验证条件并不复杂。首先，利用您尚未更动的 _静态规则_ 创建一个 `Validator` 实例：

	$v = Validator::make($data, [
		'email' => 'required|email',
		'games' => 'required|numeric',
	]);

假设我们的网页应用程序是专为游戏收藏家所设计。如果游戏收藏家收藏超过一百款游戏，我们希望他们说明为什么他们拥有这么多游戏。如，可能他们经营一家二手游戏商店，或是他们可能只是享受收集的乐趣。有条件的加入此需求，我们可以在 `Validator` 实例中使用 `sometimes` 方法。

	$v->sometimes('reason', 'required|max:500', function($input) {
		return $input->games >= 100;
	});

传递至 `sometimes` 方法的第一个参数是我们要条件式认证的字段名称。第二个参数是我们想加入验证规则。 闭包（Closure） 作为第三个参数传入，如果返回值为 true 那该规则就会被加入。这个方法可以轻而易举的建立复杂的条件式验证。您也可以一次对多个字段增加条件式验证：

	$v->sometimes(['reason', 'cost'], 'required', function($input) {
		return $input->games >= 100;
	});

注意: 


> **注意：** 传递至您的 `Closure` 的 `$input` 参数为 `Illuminate\Support\Fluent` 的实例且用来作为获取您的输入及文件的对象。

<a name="custom-error-messages"></a>
## 自定义错误信息

如果有需要，您可以设置自定义的错误信息取代默认错误信息。这里有几种方式可以设定自定义消息。

#### 传递自定义消息进验证器

	$messages = [
		'required' => 'The :attribute field is required.',
	];

	$validator = Validator::make($input, $rules, $messages);

> *注意：* 在验证中，`:attribute` 占位符会被字段的实际名称给取代。您也可以在验证信息中使用其他的占位符。

#### 其他的验证占位符

	$messages = [
		'same'    => 'The :attribute and :other must match.',
		'size'    => 'The :attribute must be exactly :size.',
		'between' => 'The :attribute must be between :min - :max.',
		'in'      => 'The :attribute must be one of the following types: :values',
	];

#### 为特定属性赋予一个自定义信息

有时您只想为一个特定字段指定一个自定义错误信息：

	$messages = [
		'email.required' => 'We need to know your e-mail address!',
	];

<a name="localization"></a>
#### 在语言包文件中指定自定义消息

某些状况下，您可能希望在语言包文件中设定您的自定义消息，而非直接将他们传递给 `Validator`。要达到这个目的，将您的信息增加至 `resources/lang/xx/validation.php` 文件的 `custom` 数组中。

	'custom' => [
		'email' => [
			'required' => 'We need to know your e-mail address!',
		],
	],

<a name="custom-validation-rules"></a>
## 自定义验证规则

#### 注册自定义验证规则

Lumen 提供了各种有用的验证规则，但是，您可能希望可以设定自定义验证规则。注册生成自定义的验证规则的方法之一就是使用 `Validator::extend` 方法：

	Validator::extend('foo', function($attribute, $value, $parameters) {
		return $value == 'foo';
	});

> **注意：** Validator 扩展应该放在 [服务提供者](/docs/providers).

自定义验证器闭包接收三个参数：要被验证的 `$attribute(属性)` 的名称，属性的值 `$value`，传递至验证规则的 `$parameters` 数组。

您同样可以传递一个类和方法到 `extend` 方法中，取代原本的闭包：

	Validator::extend('foo', 'FooValidator@validate');

注意，您同时需要为您的自定义规则制订一个错误信息。您可以使用行内自定义信息数组或是在认证语言文件里新增。

#### 扩展 Validator 类

除了使用闭包回调来扩展 Validator 外，您一样可以直接扩展 Validator 类。您可以写一个扩展自 `Illuminate\Validation\Validator` 的验证器类。您也可以增加验证方法到以 `validate` 为开头的类中：

	<?php

	class CustomValidator extends Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'foo';
		}

	}

#### 拓展自定义验证器解析器

接下来，您需要注册您自定义验证器扩展：

	Validator::resolver(function($translator, $data, $rules, $messages) {
		return new CustomValidator($translator, $data, $rules, $messages);
	});

当创建自定义验证规则时，您可能有时需要为错误信息定义自定义的占位符。您可以如上所述创建一个自定义的验证器，然后增加 `replaceXXX` 函数进验证器中。

	protected function replaceFoo($message, $attribute, $rule, $parameters) {
		return str_replace(':foo', $parameters[0], $message);
	}

如果您想要增加一个自定义信息 "replacer" 但不扩展 Validator 类，您可以使用 `Validator::replacer` 方法：

	Validator::replacer('rule', function($message, $attribute, $rule, $parameters) {
		//
	});
