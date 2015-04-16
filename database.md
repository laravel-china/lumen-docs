# 数据库使用基础

- [配置](#configuration)
- [基本使用](#basic-usage)
- [Migrations](#migrations)

<a name="configuration"></a>
## 配置

Lumen 让连接数据库和执行查找变得相当容易。数据库相关配置信息在 `.env` 文件里面, `DB_*` 开头。 你可以定义所有的数据库连接.

目前 Lumen 支持四种数据库系统： MySQL、Postgres、SQLite、以及 SQL Server。

> **注意：** 你需要在 `bootstrap/app.php`  文件里面去掉 `Dotenv::load()` 的注释才能使用 `.env` 功能。

<a name="basic-usage"></a>
## 基础使用

> **注意：** 你需要在 `bootstrap/app.php`  文件里面去掉 `$app->withFacades()` 的注释才能使用此功能。

#### 执行查找

请阅读 [Laravel 文档](http://laravel-china.org/docs/database#running-queries).

#### 查询语句构建器

请阅读 [Laravel 文档](http://laravel-china.org/docs/queries).

#### Eloquent ORM

你需要在 `bootstrap/app.php`  文件里面去掉 `$app->withEloquent()` 的注释才能使用此功能.

请阅读 [Laravel 文档](http://laravel-china.org/docs/eloquent).

<a name="migrations"></a>
## 迁移

详细文档请阅读 laravel 文档中关于 [schema builder](http://laravel-china.org/docs/schema) 以及 [migrator](http://laravel-china.org/docs/migrations).
