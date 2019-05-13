+++
showonlyimage = true
draft = false
image = "img/upload/bot-maker.png"
date = "2019-05-13T08:25:44+07:00"
title = "Integrasi Telegram Widget Login"
writer = "Randhi P. Putra"
categories = [ "code"]
weight = 1
license = "CC-by-sa License"
tags = ["laravel","php","telegram"]
+++

Assalamuallaikum, di hari Senin ini, saya coba mulai menuliskan Part 1 untuk Integrasi Laravel dengan Telegram Login. Kita mulai dari awal, anda bisa clone repo github saya atau langsung gunakan project laravel anda.
<!--more-->

Jika anda ingin memulai dari awal bisa copy paste script dibawah.

```bash

$ laravel new app
$ cd app
$ composer install
$ php artisan key:generate
$ php artisan serve --port=80
```

Laravel app baru dan fresh sudah bisa dibuka di http://127.0.0.1

## AUTH Login Laravel

Kita akan gunakan AUTH bawaan laravel, yang nantinya kita integrasikan dengan Telegram Login.

```bash
$ php artisan make:auth
$ php artisan migrate
```

Saat ini di database anda sudah ada table users, disini kita akan tambahkan kolom <code>telegram_id</code>. Buat migration script dulu ya.

```bash
$ php artisan make:migration add_telegram_id_to_user_table --table=users
```
sekarang anda edit file <code>2019_xx_xx_xxxxxx_add_telegram_id_to_user_table.php</code> yang sudah dibuat otomatis oleh artisan tadi, ada di folder <code>database/migration</code>

```php
/**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            //copy mulai disini
            $table->string('telegram_id')->after('id')->unique()->nullable();
            //copy selesai
        });
    }
```
lakukan artisan migrate, dan akan ada kolom telegram_id setelah id di table user.

Jangan lupa tambahkan <code>telegram_id</code> di fillable array di user model, supaya telegram_id bisa tersimpan.
```php
<?php

namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password', 'telegram_id'
    ];
```

## Create New User / Admin

Anda bisa create user baru dengan menggunakan seeder

```php
$ php artisan make:seed create_admin_user
```
edit <code>create_admin_user</code> di database/seeds :
```php
<?php

use Illuminate\Database\Seeder;

class create_admin_user extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        //
        \App\User::create([
            'name' => 'Admin',
            'email' => 'admin@admin.com',
            'email_verified_at' => now(),
            'password' => bcrypt('admin'),
            'telegram_id' => '12345678910'
        ]);
    }
}

```
ganti telegram id dengan id telegram anda. Klik [disini](../telegram-id) untuk tahu cara mendapatkan telegram id.

tambahkan class seeder di database seeder :
```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $this->call(create_admin_user::class);
    }
}

```
jalankan seeder
```bash
$ php artisan migrate --seed
```
Anda sudah bisa login dengan username: <code>admin@admin.com</code> dan pasw: <code>admin</code>


## Buat Bot Baru di Telegram

![bot-father](/img/upload/bot-maker.png "botfather")

Mari kita buat dulu bot di Telegram 
- chat [@botfather](https://t.me/botfather) dan ketik <code>/newbot</code>
- beri nama misal: <code>JDigital_bot</code>
- simpan token yg diberikan dari botfather.
- contoh token formatnya seperti ini <code>715028267:AAGkzm2y-HFiZrsEu4IKEhVq2Ls-0GnfprI</code>

####Set Domain di bot yang baru anda buat

Jangan lupa berikan domain di bot yang baru anda buat, atau widget tidak akan bisa muncul.

- <code>/setdomain</code> dan balas dengan 127.0.0.1


## Install module Telegram Login dengan composer

Oya untuk tetap menjunjung konsep programing DRY "Don't Repeat Yourself", disini saya menggunakan module milik [azate](https://github.com/azate/laravel-telegram-login-auth "gihub-link").

Kembali ke laravel root folder dan require dengan composer.

```bash
$ composer require azate/laravel-telegram-login-auth
```
publish config 
```bash
$ php artisan vendor:publish --provider="Azate\LaravelTelegramLoginAuth\TelegramLoginServiceProvider"
```
Masukkan token dari botfather di <code>/config/telegram_login_auth.php</code>
```php
<?php

return [
    'token' => env('TELEGRAM_LOGIN_AUTH_TOKEN', '715028267:AAGkzm2y-HFiZrsEu4IKEhVq2Ls-0GnfprI'),
];

```

Atau bisa juga input token di <code>.env</code>
```php
APP_DEBUG=true
APP_URL=http://localhost

TELEGRAM_LOGIN_AUTH_TOKEN=715028267:AAGkzm2y-HFiZrsEu4IKEhVq2Ls-0GnfprI
```

## Login Controller

Beralih ke Login Controller. Ada perbedaan dari doc resmi di github milik azate menggunakan AuthController. Laravel App di tutorial ini menggunakan versi Laravel 5.8  make:Auth yang menggunakan LoginController. Jadi ada beberapa penyesuaian yang harus kita lakukan, edit <code>/app/Http/Controllers/Auth/LoginController.php</code> :

```php
<?php

namespace App\Http\Controllers\Auth;

use App\User;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Azate\LaravelTelegramLoginAuth\TelegramLoginAuth;

use Auth;
use Session;

class LoginController extends Controller
{
    
    protected $telegram;

    /**
     * Get user info and log in (hypothetically)
     *
     * @return \Illuminate\Routing\Redirector|\Illuminate\Http\RedirectResponse
     */
    public function handleTelegramCallback()
    {
        if ($this->telegram->validate()) {
            $user = $this->telegram->user();

            $authUser = User::where('telegram_id',$user['id'])->first();
            
            if ($authUser){
                Auth::login($authUser, true);
                return redirect('/home');
            } else {
                Session::flush();
                Session::put('data', $user);
                Session::put('_from', 'telegram');
                return redirect('/register');
            }
            
        }

       
    }
    
    /*
    |--------------------------------------------------------------------------
    | Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles authenticating users for the application and
    | redirecting them to your home screen. The controller uses a trait
    | to conveniently provide its functionality to your applications.
    |
    */

    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = '/home';

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct(TelegramLoginAuth $telegram)
    {   
        $this->telegram = $telegram;
        $this->middleware('guest')->except('logout');
    }
}
```

## Handle callback dari Telegram

Edit <code>route/web.php</code>

```php
<?php

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/
Route::get('/', 'WelcomeController@index')->name('welcome');

Route::get('auth/telegram/callback', 'Auth\LoginController@handleTelegramCallback')->name('auth.telegram.handle');
 

```

## Tambahkan widget Login dari Telegram di welcome page

Dapatkan kode snippet untuk widget login [https://core.telegram.org/widgets/login](https://core.telegram.org/widgets/login)

![bot-widget](/img/upload/bot_widget.png)

Jangan lupa set <code>Authorization Tipe</code> ke Redirect to URL : http://127.0.0.1/auth/telegram/callback

tambahkan di <code>/resources/views/welcome.blade.php</code>
```html
<div class="content">
                <div class="title m-b-md">
                    Laravel
                </div>

                <script async src="https://telegram.org/js/telegram-widget.js?5" data-telegram-login="jdigital_bot" data-size="medium" data-auth-url="http://127.0.0.1/auth/telegram/callback" data-request-access="write"></script>
                
            </div>
```

Halaman welcome anda akan berubah menjadi seperti ini

![welcome](/img/upload/Screenshot_2019-05-13-Laravel.png)

Silahkan anda coba click login with telegram, input no hp anda, dan confirm/approved security access via telegram anda.

Kalau sukses anda akan di-redirect ke home :

![success](/img/upload/redirect-home.png)

![/](/img/upload/root-view.png)

Tampilan di atas ada saat anda kembali ke / (root)

;) success? harus sukses. Kalau ada kendala, komen dibawah ya.

clone git repo tutorial ini di :
```
$ git clone https://github.com/randhipp/JD-laravel-telegram-part-1.git
```


>Postingan ini dibawah lisensi [Creative Commons Attribution ShareAlike](https://creativecommons.org/licenses/by-sa/3.0/us/deed.id "license"), anda bisa share, translate, dan remix dengan mencantumkan link asli ke post ini.

<img src="https://upload.wikimedia.org/wikipedia/commons/2/2f/CC_BY-SA_3.0.png" width="80">