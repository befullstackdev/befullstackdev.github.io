---
layout: post
title: 'Laravel TokenMismatchException 错误排查'
category: laravel
comments: true
finished: true
---

近来学习 Laravel 时,时不时会遇到

```php 
TokenMismatchException in VerifyCsrfToken.php file...
```

解决过两次,每次都有不一样的原因, 故来个小结吧.

常见的原因

1. 表单中缺少 `<input type="hidden" name="_token" value="{ { csrf_token() } }">`
2. 项目迁移所有文件直接 copy 了或者更换了新域名(本地经常配置 virtualhost)

解决方案

1. 针对 `1` 的情况, 最简单,表单中添加即可;
2. 针对 `2` 的情况, 重新生成 `APP_KEY`, `php artisan key:generate`


补充:

`config/session.php` 中的`domain`是否正确, 我有一次就是忘记了这个配置更改成当前域名.建议在 env 中设置 `SESSION_DOMAIN`, 然后在`config/session.php`中这样改:

```php
'domain' => env('SESSION_DOMAIN', 'localhost')
```
