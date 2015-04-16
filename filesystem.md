# 文件系统 / 云存储

- [简介](#introduction)
- [配置文件](#configuration)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 简介

Lumen 有很棒的文件系统抽象层，是基于 Frank de Jonge 的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。 Lumen 集成的 Flysystem 提供了简单的接口，可以操作本地端空间、 Amazon S3 、 Rackspace Cloud Storage 。更好的是，它可以非常简单的切换不同保存方式，但仍使用相同的 API 操作！

<a name="configuration"></a>
## 配置文件

文件系统的配置文件放在 `config/filesystems.php` 。在这个文件内你可以配置所有的「硬盘」。每个硬盘代表一种保存方式和地点。默认的配置文件内已经包含了所有保存方式的例子。所以只要修改保存配置和认证即可！

在使用 S3 或 Rackspace 之前，你必须先用 Composer 安装相对应的扩展包：

- Amazon S3: `league/flysystem-aws-s3-v2 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

当然，你可以加入任意数量的硬盘配置文件，甚至配置多个硬盘都使用同一种保存方式。

使用本地端空间时，要注意所有的操作路径都是相对于配置文件里 `local` 的 `root` ，默认的路径是 `storage/app` 。因此下列的操作将会保存一个文件在 `storage/app/file.txt` ：

	Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## 基本用法

> **Note:** 如果你想使用 `Storage` facade 的话, 请在 `bootstrap/app.php` 文件里面把 `$app->withFacades()`去掉注释. 

可以用 `Storage` facade 操作所有写在配置文件里的硬盘。或者是，你也可以将 `Illuminate\Contracts\Filesystem\Factory` 类型提示写到任何类里，经由 Lumen 的 [服务容器](/docs/5.0/container) 解析。

#### 取得一个特定硬盘

	$disk = Storage::disk('s3');

	$disk = Storage::disk('local');

#### 确认文件是否存在

	$exists = Storage::disk('s3')->exists('file.jpg');

#### 使用默认硬盘调用方法

	if (Storage::exists('file.jpg'))
	{
		//
	}

#### 取得文件内容

	$contents = Storage::get('file.jpg');

#### 配置文件内容

	Storage::put('file.jpg', $contents);

#### 加入内容到文件开头

	Storage::prepend('file.log', 'Prepended Text');

#### 附加内容到文件结尾

	Storage::append('file.log', 'Appended Text');

#### 删除文件

	Storage::delete('file.jpg');

	Storage::delete(['file1.jpg', 'file2.jpg']);

#### 复制文件到新的路径

	Storage::copy('old/file1.jpg', 'new/file1.jpg');

#### 移动文件到新的路径

	Storage::move('old/file1.jpg', 'new/file1.jpg');

#### 取得文件大小

	$size = Storage::size('file1.jpg');

#### 取得最近修改时间 (UNIX)

	$time = Storage::lastModified('file1.jpg');

#### 取得目录下所有文件

	$files = Storage::files($directory);

	// Recursive...
	$files = Storage::allFiles($directory);

#### 取得目录下所有子目录

	$directories = Storage::directories($directory);

	// Recursive...
	$directories = Storage::allDirectories($directory);

#### 建立目录

	Storage::makeDirectory($directory);

#### 删除目录

	Storage::deleteDirectory($directory);
