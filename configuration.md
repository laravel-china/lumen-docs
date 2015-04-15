# 配置

- [介绍](#introduction)
- [安装后的配置](#after-installation)
- [配置文件](#configuration-files)
- [优雅链接](#pretty-urls)

<a name="introduction"></a>
## 介绍


与 Laravel 不同的是, Lumen 只使用单一的 `.env`  配置文件, 你阅读下 `.env.example` 文件, 看下有哪些选项需要配置.

> **注意:** 如果你想要使用 `vlucas/phpdotenv` 库来加载你的环境变量, 请把 `bootstrap/app.php` 文件里面的 `Dotenv::load` 这个调用注释掉.

<a name="after-installation"></a>
## 安装后的配置

Lumen 基本上不需要配置, 但是, 安装后的第一件事情是在 `.env` 文件里面设置你的 `APP_KEY`, 此值应该为随机的 32 位字符串.

你可以查看其他的文档, 如下:

- [缓存](/docs/cache#configuration)
- [数据库](/docs/database#configuration)
- [队列](/docs/queues#configuration)
- [会话](/docs/session#configuration)


> **注意:** 请注意千万不要在生产环境下把 `APP_DEBUG` 选项设置为 `true`。

<a name="permissions"></a>
### 权限

Lumen 框架有一个目录需要额外配置权限: `storage` 目录要让服务器有写入的权限。

<a name="configuration-files"></a>
## 配置文件

默认情况下, Lumen 使用单一的 `.env` 文件来配置你的应用, 然而, 你也可以使用 `Laravel 风格` 的配置方法。只需要把 `vendor/laravel/lumen-framework/config` 文件夹下对应的配置文件复制到根目录下的 `config` 文件里面就行. 

使用这种方法, 能有更多的配置灵活度. 

#### 自定义配置文件

你可以创建自定义的配置文件, 并使用 `$app->configure()` 方法进行加载. 

举个栗子, 把自定义配置文件放置于 `config/options.php`, 可以这样加载:

	$app->configure('options');

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
