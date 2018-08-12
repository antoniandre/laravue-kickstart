# How to get here from scratch

Steps to scaffold Laravel 5.6 with JWT authentication, register, login, logout.

## Create Laravel project

```
laravel new laravue-kickstart
```

If you don't have Laravel installer, install it from composer then add in path.

```
composer global require "laravel/installer"
set PATH $PATH ~/.composer/vendor/bin
```

## Fix database issue

edit App/Providers/AppServiceProvider.php:

Import Schema:

```
use Illuminate\Support\Facades\Schema;
```

In `boot()` function write:

```
Schema::defaultStringLength(191);
```