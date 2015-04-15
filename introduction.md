# Introduction

- [什么是 Lumen?](#what-is-lumen)
- [在什么时候使用 Lumen?](#when-should-i-use-lumen)
- [Lumen 包含了哪些 Laravel 的功能?](#lumen-features)

<a name="what-is-lumen"></a>
## 什么是 Lumen?

Lumen 是一个由 Laravel 元件搭建而成的微框架, 由 Laravel 官方维护. Lumen 为速度而生, 是当前最快的 PHP 框架之一, 甚至比类似的微框架 `Silex` 速度还要快.

Lumen 比其他微框架的优点是, 构建在 Laravel 之上, 使其具备 Laravel 强大的功能, 如 路由, 依赖注入, Eloquent ORM, 数据库迁移管理, 队列和计划任务等.

Laravel 本来就是一个功能齐全, 速度飞快的框架, 但是 Lumen 因为去除了很多 Laravel 的配置和可自定义的选项, 速度越加飞快, 毫秒必争. 

飞快的速度, 再加上 Laravel 非常方便的功能, 使用 Lumen 开发应用会是非常愉悦的体验.

<a name="when-should-i-use-lumen"></a>
## 在什么时候使用 Lumen?

Lumen 专为微服务或者 API 设计, 举个例子, 如果你的应用里面有部分业务逻辑的请求频率比较高, 就可以单独把这部分业务逻辑拿出来, 使用 Lumen 来构建一个小 App. 

因为 Lumen 是对 Laravel 优化了框架的加载机制, Lumen 对资源的要求好少很多. 

当然, 你可以使用 队列系统 与你的主 Laravel 应用进行交互. Laravel 和 Lumen 从一开始就是设计成能一起很好的工作, 并且, 配合使用, 允许你构架一个强大的, 以微服务为驱动的应用程序. 

Lumen 同时也非常适用于构建 API 接口, 此类型的应用通常情况下不需要具备 `全栈框架` 的所有功能, 如 HTTP 会话管理, Cookies, 和模版系统. 

### Lumen 的限制

因为对框架的加载进行了优化, 去除灵活性来换取速度, 所以 Lumen 的可自定义性不是很强, 一些专为 Laravel 开发的扩展包可能无法使用, 如开发者工具条, CMS 系统等.

Lumen 没有使用 Symfony 的路由模块, 而是采用了速度更加快的 `nikic/fast-route`. 如果你需要使用 Symfony 的路由功能, 如 子域名等高级路由功能, Lumen 可能不适合你, 建议使用功能更加齐全的 Laravel.

如果你真的选择了全栈框架, 请放心使用, 构建在 Laravel 上的应用程序能处理每天 15,000,000 以上的请求, 没什么可担忧的.

<a name="lumen-features"></a>
## Lumen 包含了哪些 Laravel 的功能

Lumen 包含了大部分的 Laravel 全栈框架的功能: 

- Blade 模版引擎
- Caching 缓存系统
- Command Scheduler 计划任务
- Controllers 控制器
- Eloquent ORM 数据库操作
- Error Handling 错误处理
- Database Abstraction 数据库抽象层
- Dependency Injection 依赖注入
- Logging 日志系统
- Queued Jobs 队列系统

Lumen 独特的初始化机制, 使其在功能强大的同时, 具备了高性能, 是构建微服务架构应用的绝佳方案.

请随意查阅各个功能对应的文档.

