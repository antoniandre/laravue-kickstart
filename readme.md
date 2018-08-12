# How to get here from scratch

Steps to scaffold Laravel 5.6 with JWT authentication, register, login, logout.

## Create Laravel project & virtual host

```
laravel new laravue-kickstart
```

If you don't have Laravel installer, install it from composer then add in path:

```
composer global require "laravel/installer"
set PATH $PATH ~/.composer/vendor/bin
```

You can also create project directly from composer:

```
composer create-project --prefer-dist laravel/laravel laravue-kickstart
```

Create virtual host in `/Applications/MAMP/conf/apache/extra/httpd-vhosts.conf`

```
<VirtualHost *:80>
  DocumentRoot "/Applications/MAMP/htdocs/laravue-kickstart/public"
  ServerName laravue.test
</VirtualHost>
```

Add your virtual host to `/etc/hosts`:

```
127.0.0.1       laravue.test
```

Then reboot your Apache server.

Check in the browser your project is accessible at http://laravue.test

## Set your environment in `.env`

Create a new empty database from phpMyAdmin or terminal
Then add the database details in .env file

## Fix database issue & migrate

edit App/Providers/AppServiceProvider.php:

Import Schema:

```
use Illuminate\Support\Facades\Schema;
```

In `boot()` function write:

```
Schema::defaultStringLength(191);
```

After that you can migrate the user/password tables in database.

```
php artisan migrate
```