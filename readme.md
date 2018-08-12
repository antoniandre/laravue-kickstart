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

```php
use Illuminate\Support\Facades\Schema;
```

In `boot()` function write:

```php
Schema::defaultStringLength(191);
```

After that you can migrate the user/password tables in database.

```
php artisan migrate
```


## Add JWT Authentication

```
composer require tymon/jwt-auth:dev-develop --prefer-source
```

Then edit `config/app.php`:

Append `Tymon\JWTAuth\Providers\LaravelServiceProvider::class` to the list of providers

Append `'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class,` to the list of aliases

In `app/Http/Kernel.php` add this to the list of routeMiddleware:

```php
  'jwt.auth' => \Tymon\JWTAuth\Middleware\GetUserFromToken::class,
  'jwt.refresh' => \Tymon\JWTAuth\Middleware\RefreshToken::class,
```

Now to publish provider changes run:

```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"
php artisan jwt:secret
```


## Update User model

Import JWT and add methods.

```
use Tymon\JWTAuth\Contracts\JWTSubject;
```

```php
class User extends Authenticatable implements JWTSubject
...

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
```


## Add auth controller and form

create __app/Http/Controllers/AuthController.php__

```
php artisan make:controller AuthController
```

add imports:

```php
use App\User;
use Auth;
use JWTAuth;
```

Fill class content:

```php
  public function register(RegisterFormRequest $request)
  {
      $user = new User;
      $user->email = $request->email;
      $user->name = $request->name;
      $user->password = bcrypt($request->password);
      $user->save();
      return response([
          'status' => 'success',
          'data' => $user
      ], 200);
  }

  public function login(Request $request)
  {
      $credentials = $request->only('email', 'password');
      if ( ! $token = JWTAuth::attempt($credentials)) {
              return response([
                  'status' => 'error',
                  'error' => 'invalid.credentials',
                  'msg' => 'Invalid Credentials.'
              ], 400);
      }
      return response([
              'status' => 'success'
          ])
          ->header('Authorization', $token);
  }

  public function user(Request $request)
  {
      $user = User::find(Auth::user()->id);
      return response([
              'status' => 'success',
              'data' => $user
          ]);
  }

  public function refresh()
  {
      return response([
              'status' => 'success'
          ]);
  }

  public function logout()
  {
      JWTAuth::invalidate();
      return response([
              'status' => 'success',
              'msg' => 'Logged out Successfully.'
          ], 200);
  }
```

Create __App/Http/Requests/RegisterFormRequest.php__

```
php artisan make:request RegisterFormRequest
```

Then set `authorize()` method return to true and fill in the rules:

```php
return [
    'name' => 'required|string|unique:users',
    'email' => 'required|email|unique:users',
    'password' => 'required|string|min:6|max:10',
];
```


## Add front end router and authentication

```
npm install
npm i --d vue-router vue-axios @websanova/vue-auth
```

Replace content of app.js:

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import axios from 'axios';
import VueAxios from 'vue-axios';
import App from './App.vue';
import Dashboard from './components/Dashboard.vue';
import Home from './components/Home.vue';
import Register from './components/Register.vue';
import Login from './components/Login.vue';

Vue.use(VueRouter);
Vue.use(VueAxios, axios);

axios.defaults.baseURL = 'http://laravel.test/api';

const router = new VueRouter({
    routes: [{
        path: '/',
        name: 'home',
        component: Home
    }, {
        path: '/register',
        name: 'register',
        component: Register,
        meta: {
            auth: false
        }
    }, {
        path: '/login',
        name: 'login',
        component: Login,
        meta: {
            auth: false
        }
    }, {
        path: '/dashboard',
        name: 'dashboard',
        component: Dashboard,
        meta: {
            auth: true
        }
    }]
});

Vue.router = router
Vue.use(require('@websanova/vue-auth'), {
    auth: require('@websanova/vue-auth/drivers/auth/bearer.js'),
    http: require('@websanova/vue-auth/drivers/http/axios.1.x.js'),
    router: require('@websanova/vue-auth/drivers/router/vue-router.2.x.js'),
});

App.router = Vue.router
new Vue(App).$mount('#app');
```

## Make the front end work

Create `App.vue` file

```html
<template>
    <div class="panel panel-default">
        <div class="panel-heading">
            <nav>
                <ul class="list-inline">
                    <li>
                        <router-link :to="{ name: 'home' }">Home</router-link>
                    </li>
                    <li v-if="!$auth.check()" class="pull-right">
                        <router-link :to="{ name: 'login' }">Login</router-link>
                    </li>
                    <li v-if="!$auth.check()" class="pull-right">
                        <router-link :to="{ name: 'register' }">Register</router-link>
                    </li>
                    <li v-if="$auth.check()" class="pull-right">
                        <a href="#" @click.prevent="$auth.logout()">Logout</a>
                    </li>
                </ul>
            </nav>
        </div>
        <div class="panel-body">
            <router-view></router-view>
        </div>
    </div>
</template>
```

Update `welcome.blade.php`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>Laravel</title>

    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">

</head>
<body>
    <div class="container">
        <div id="app"></div>
    </div>
    <script src="/js/app.js"></script>
</body>
</html>
```

### Create Home.vue Vue components

```
<template>
  <h1>Laravel 5 Vue SPA Authentication</h1>
</template>
```

### Add Login, Register and Dashboard Vue components

__`Register.vue`__

```html
<template>
    <div>
        <div class="alert alert-danger" v-if="error && !success">
            <p>There was an error, unable to complete registration.</p>
        </div>
        <div class="alert alert-success" v-if="success">
            <p>Registration completed. You can now <router-link :to="{name:'login'}">sign in.</router-link></p>
        </div>
        <form autocomplete="off" @submit.prevent="register" v-if="!success" method="post">
            <div class="form-group" v-bind:class="{ 'has-error': error && errors.name }">
                <label for="name">Name</label>
                <input type="text" id="name" class="form-control" v-model="name" required>
                <span class="help-block" v-if="error && errors.name">{{ errors.name }}</span>
            </div>
            <div class="form-group" v-bind:class="{ 'has-error': error && errors.email }">
                <label for="email">E-mail</label>
                <input type="email" id="email" class="form-control" placeholder="user@example.com" v-model="email" required>
                <span class="help-block" v-if="error && errors.email">{{ errors.email }}</span>
            </div>
            <div class="form-group" v-bind:class="{ 'has-error': error && errors.password }">
                <label for="password">Password</label>
                <input type="password" id="password" class="form-control" v-model="password" required>
                <span class="help-block" v-if="error && errors.password">{{ errors.password }}</span>
            </div>
            <button type="submit" class="btn btn-default">Submit</button>
        </form>
    </div>
</template>
```

__`login.vue`__

```html
<template>
    <div>
        <div class="alert alert-danger" v-if="error">
            <p>There was an error, unable to sign in with those credentials.</p>
        </div>
        <form autocomplete="off" @submit.prevent="login" method="post">
            <div class="form-group">
                <label for="email">E-mail</label>
                <input type="email" id="email" class="form-control" placeholder="user@example.com" v-model="email" required>
            </div>
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" class="form-control" v-model="password" required>
            </div>
            <button type="submit" class="btn btn-default">Sign in</button>
        </form>
    </div>
</template>
```

__`Dashboard.vue`__

```html
<template>
    <h1>Laravel 5 – Our Cool Dashboard</h1>
</template>
```


## Create api routes

Add the routes in `routes/api.php`

```php
Route::post('auth/register', 'AuthController@register');
Route::post('auth/login', 'AuthController@login');
Route::group(['middleware' => 'jwt.auth'], function(){
	Route::get('auth/user', 'AuthController@user');
	Route::post('auth/logout', 'AuthController@logout');
});
Route::group(['middleware' => 'jwt.refresh'], function(){
	Route::get('auth/refresh', 'AuthController@refresh');
});
```

## Make Ajax calls to API from Front end

Add axios AJAX call from `register.vue`

```js
<script>
    export default {
        data(){
            return {
                name: '',
                email: '',
                password: '',
                error: false,
                errors: {},
                success: false
            };
        },
        methods: {
            register(){
                var app = this
                this.$auth.register({
                    data: {
                        name: app.name,
                        email: app.email,
                        password: app.password
                    },
                    success: function () {
                        app.success = true
                    },
                    error: function (resp) {
                        app.error = true;
                        app.errors = resp.response.data.errors;
                    },
                    redirect: null
                });
            }
        }
    }
</script>
```

Add axios AJAX call from `login.vue`

```js
<script>
  export default {
    data(){
      return {
        email: null,
        password: null,
        error: false
      }
    },
    methods: {
      login(){
        var app = this
        this.$auth.login({
            params: {
              email: app.email,
              password: app.password
            },
            success: function () {},
            error: function () {},
            rememberMe: true,
            redirect: '/dashboard',
            fetchUser: true,
        });
      },
    }
  }
</script>
```