# 安装

- [安装 Composer](#install-composer)
- [安装 Lumen](#install-lumen)
- [环境需求](#server-requirements)

<a name="install-composer"></a>
## 安装 Composer

Lumen 使用 [Composer](http://getcomposer.org) 来管理依赖扩展包, 所以, 在使用 Lumen 之前, 你需要先安装 Composer.

<a name="install-lumen"></a>
## 安装 Lumen

### 通过 Lumen 命令安装

首先, 使用下面命令下载 Lumen 命令: 

	composer global require "laravel/lumen-installer=~1.0"

请确定把 `~/.composer/vendor/bin` 路径放置于您的 PATH 里， 这样 Lumen 执行文件就会存在你的系统。

一旦安装完成后，就可以使用 `lumen new` 命令建立一份全新安装的 Lumen 应用，例如： `laravel new service` 将会在当前目录下建立一个名为 service 的目录， 此目录里面存放着全新安装的 Lumen 相关代码，此方法跟其他方法不一样的地方在于会提前安装好所有相关代码，不需要再通过 composer install 安装相关依赖，速度会快许多。

	lumen new service

### 通过 Composer 创建项目命令

你一样可以通过 Composer 在命令行执行 create-project 来安装 Lumen:

	composer create-project laravel/lumen --prefer-dist

<a name="server-requirements"></a>
## 环境需求

Lumen 框架有一些系统上的需求：

- PHP >= 5.4
- Mcrypt PHP 扩展
- OpenSSL PHP 扩展
- Mbstring PHP 扩展
- Tokenizer PHP 扩展

<a name="configuration"></a>
## 配置信息

Lumen 几乎不需多余的配置就可以马上使用。你可以马上着手开发！然而，你可以查看其他的文档, 如下:

- [缓存](/docs/cache#configuration)
- [数据库](/docs/database#configuration)
- [队列](/docs/queues#configuration)
- [会话](/docs/session#configuration)

<a name="permissions"></a>
### 权限

Lumen 框架有一个目录需要额外配置权限: `storage` 目录要让服务器有写入的权限。

<a name="pretty-urls"></a>
## 优雅链接

### Apache

Lumen 框架通过 public/.htaccess 文件来让网址中不需要 index.php。如果你的网页服务器是使用 Apache 的话，请确认是否有开启 mod_rewrite 模块。

假设 Laravel 附带的 .htaccess 文件在 Apache 无法生效的话，请尝试下面的方法：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

Nginx 上，在配置中增加下面的配置，就可以使用「优雅链接」了: 

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

当然，如果你使用 [Homestead](http://laravel.com/docs/homestead) 的话，优雅链接会自动的帮你配置完成。