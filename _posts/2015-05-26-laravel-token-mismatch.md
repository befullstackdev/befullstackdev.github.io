---
layout: post
title: Laravel TokenMismatchException 错误排查
category: laravel
comments: true
finished: true
---

近来学习 Laravel 时,时不时会遇到 ```TokenMismatchException in VerifyCsrfToken.php file...```, 解决过几次,每次都有不一样的原因, 故来个小结吧.

常见的原因

1. 表单中缺少 `<input type="hidden" name="_token" value="{{ csrf_token() }}">`
2. 项目迁移所有文件直接 copy 了或者更换了新域名(本地经常配置 virtualhost)

解决方案

针对 `1` 的情况, 表单中添加即可;

针对 `2` 的情况, 重新生成 `APP_KEY`, ` php artisan key:generate`

