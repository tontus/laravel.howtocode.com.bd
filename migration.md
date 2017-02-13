#মাইগ্রেশন
***
লারাভেল মাইগ্রেশন(কিছু ফাইল এর মধ্যে কিছু ক্লাস)এর মাধ্যমে জানে আমরা ডাটাবেজ এ কি কি করছি। অর্থাৎ যখন আমরা টেবিল বানাই, কোনও ফিল্ড এর নাম পালটাই সবই আমরা এই মাইগ্রেশন এর ভিতর লিপিবদ্ধ করি।

বই এর ভাষায় বলতে গেলে মাইগ্রেশন(Migration) হলো আমাদের এপ এর ডাটাবেজ(Database) এর স্কিমা(Schema).

#### সুবিধাঃ
কোড লেখার সময় আমরা নানা রকম ভার্শন কন্ট্রোল ব্যবহার করি, যেমন git. এর সুবিধা আমরা সবাই জানি যে, দলের সবাই আমাদের কাজের পরিবর্তন দেখছে, প্রয়োজনে যেকোনো কাজ আমরা undo করে আগের যেকোনো পর্যায়ে সহজে জেতে পারি। লারাভেল এর মাইগ্রাশন ব্যবহার করে কোড এর মতো ডাটাবেজ এও এই শুবিধা নিতে পারি। অর্থাৎ মাইগ্রেশন হলো অনেকটা ডাটাবেজ এর ভার্শন কন্ট্রোল সিস্টেম।
শুধু তাই না, লারাভেল কে ```migrate``` করতে বললে লারাভেল আমাদের তৈরি ```migration``` এর উপর ভিত্তি করে আমাদের ডাটাবেজ এ সব টেবিল, সব ফিল্ড, সব পরিবর্তন-পরিবর্ধন এবং আরও যা যা থাকবে সব ঠিক ঠিক তৈরি করে দিবে।
ধরুন, একটি ডেভেলপার দল git ব্যবহার করে একটি প্রোজেক্ট করছে। হঠাৎ একজনের মনে হলো user টেবিল এ নতুন একটি ফিল্ড লাগবে যার নাম active এবং boolean টাইপ। এবং তিনি migration ব্যবহার করে টেবিলের পরিবর্তন টা করলেন ও git এ পাঠিয়ে দিলেন। অন্যান্য সদস্যরা git merge করলেন এবং দেখলেন যে নতুন ```migration``` আছে এবং উনি লারাভেলকে বলেন ```migrate``` কর, জাদুর মতো তার ডাটাবেজও আপডেট হয়ে গেল। আগের মতো আর ```newdbstracture.sql``` মত ফাইল গুলো আর  চালা চালি করা লাগবে না।

**মাইগ্রাশন নিয়ে আরও বিস্তারিত আলোচনা করবো কিন্তু তার আগে আসেন আমরা আমাদের লারাভেল প্রোজেক্ট এর সাথে একটি ডাটাবেজ এর সংযোগ দেই।**

সাধারণ ভাবে আমরা সবাইই কম বেশি mySQL ব্যবহার করি তাই আমার উদাহরনে আমি mySQLই ব্যবহার করলাম।

আসুন যার যার এনভায়রনমেন্ট আনুসারে mydb নামে একটি ডাটাবেজ বানাই।
ধরে নেই,

Database Host: 127.0.0.1

Database Name: mydb

User: root

Password: secret

এই বার আমাদের প্রোজেক্ট ফোল্ডার এর রুটে ```.env``` নামে ফাইলটা খুলে নিম্ন লিখিত অংশের মতো পরিবর্তন আনি।

```php
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=mydb
DB_USERNAME=root
DB_PASSWORD=secret
```
আপনি নিশ্চয় বুজতে পারছেন এগুলা constant variable, কিন্তু কোথায় এগুলার ব্যবহার হয়??

মনে আছে আমরা **[ইন্সটলেশন অধ্যায়ে](http://laravel.howtocode.com.bd/installation.html)** লারাভেল ৫.৩ ফাইল বিন্যাস দেখেছিলাম?

হুম.. তাহলে আসুন ```/config/database.php``` ফাইলটি খুলি।

বাহ সব পরিষ্কার তাই না? তাও একটি লাইন এর মানে দেখিঃ

```php
'database' => env('DB_DATABASE', 'forge'),
```
```.env``` ফাইল এর ভেতর যদি ```DB_DATABASE``` নামের ভারিয়াবল এ কোনও value সেট করি তাহলে database হবে ওই value টি নয়তো database এর value হবে ```forge``` ।

তাহলে আমাদের এপ এর সাথে ডাটাবেজ চলে আসলো!!!

**এবার আসুন আমরা আবার মাইগ্রেশনে ফিরে আসি।**

মাইগ্রেশনের ফাইল গুলি থাকে ``` /database/migrations ``` ফোল্ডার এ। ফোল্ডারটি খুললে আমরা নিচের ফাইলটির মতো একটি(আসলে লারাভেল আমাদের জন্য দুটি ফাইল আগেই তৈরি করে রাখে) দেখতে পাবো।

```
2014_10_12_000000_create_users_table.php

```

সংখ্যা গুলো দিয়ে ফাইলটি কবে ও কখন তৈরি করা হয়েছে সেটা দেখায়, পরের অংশে মাইগ্রেশনটিতে কি করা হয়েছে সেটা থাকে। নামটি পরলেই আমরা বুজতে পারিঃ এটা দিয়ে users নামে একটি টেবিল তৈরি করা হয়েছে। এবং এটি একটি PHP ফাইল, হা হা হা ।

আসুন ফাইলটি খুলি ও একটু বুঝে নেই কারণ কিছু দিনের মধ্যেই আমরা এরকম অনেক ফাইল বানাবো।

```php
<?php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{    
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('users');
    }
}
```
প্রথমত Blueprint ও Migration নামের দুটি ক্লাসকে ব্যবহার করা হবে।
Migration নামের ক্লাস কে এক্সটেন্ড করে CreateUsersTable নামের একটি ক্লাস লেখা হয়েছে।

এই ক্লাসে আমরা দুটি মেথড দেখতে পাই, একটি up অন্যটি down ।
**up** মেথড দিয়ে আমরা কোনও একটি কাজ করবো যেমন এখানে টেবিল তৈরি করা হয়েছে। এই মেথড ব্যবহার করে আমরা কোন নতুন ফিল্ড একটি টেবিলে যোগ করতে পারি, কখনো নাম পরিবর্তন করতে পারি বা অন্য কোনও পরিবর্তনও করতে পারি।
আর **down** মেথড এ আমারা up মেথডে যে পরিবর্তন টা করেছি সেটা কিভাবে undo করা যায় সেই প্রসেসটা বলে দিবো। এখানে একটা টেবিল বানানো হয়েছিল তাই down মেথডে এটাকে উধাও করে দেবার প্রসেসটা বলা হয়েছে।

**up** মেথড এর ভেতর লারাভেল এর স্কিমা(Schema) ফ্যাসাদ(Facade) ব্যবহার করে টেবিলটি বানানো হয়েছে।

**চলুন না আমরাও একটি টেবিল এর স্কিমা(Schema) বানিয়ে ফেলিঃ**

ভয় পেলেন নাকি যে আবার কত্ত গুলো কোড লেখা লাগবে? ভয় নেই, লারাভেল আমাদের জন্য শক্তিশালী একটি CLI(Command Line Interface) দিয়েছে। আসুন ব্যবহার করি।
Terminal এ আমাদের প্রজেক্ট এর ভেতর ঢুকে নিচের কমান্ড টি run করাইঃ
```bash
php artisan make:migration create_posts_table --create=posts
```
নিশ্চয় দেখবেন
```bash
Created Migration: 2016_04_18_201310_create_posts_table
```
তবে প্রথমের দিন ও সময় টি আপনার হবে।

এবার আপনার Code Editor এ এই ```/database/migrations``` ডিরেক্টরি খুলুন ও নতুন তৈরি করা migration টি দেখুন(প্রয়োজনে একবার path টি refresh করে নিন)

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePostsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->increments('id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('posts');
    }
}
```
আমরা আগেই সাধারণ একটি মাইগ্রেশনের ব্যাখ্যা পড়েছি, তাই সরাসরি up মেথড এ চলে আসি।

up মেথড এর ভিতরের অংশটুকু কে যদি আমি বাংলা ভাষায় বলার চেষ্টা করি তাহলেঃ
Schema ফ্যাসাদ, টেবিল তৈরি(create) করো যার নাম posts এবং টেবিলটির গঠনটি(Blueprint) এরকম যে এর ভিতর একটি id ফিল্ড থাকবে যেটা auto increment হবে এবং timestamps() এর মাধ্যমে আরও দুটি ফিল্ড বানাও created_at ও updated_at নামে যা দিন ও সময় সংরক্ষণ করবে।

সাধারনত একটি টেবিলে একটি ID লাগে এবং ওই রেকর্ডটি কবে তৈরি ও পরিবর্তন হয়েছে সেইটাও জানতে হয়, তাই লারাভেল আগেই আমাদের জন্য লিখে দিয়েছে। কিন্তু এগুলো আমাদের প্রয়োজন না হোলে মুছে ফেলতে পারি। যাক, আমাদের কাল্পনিক posts টেবিলটায় আরও কিছু ফিল্ড/কলাম(Column) যোগ করিঃ

```php
Schema::create('posts', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title', 255);
            $table->text('content');
            $table->string('slug')->nullable();
            $table->tinyInteger('status')->default(1);
            $table->timestamps();
        });
```
বুজতেই পারছেন আমি title নামের string টাইপের ২৫৫ অক্ষরের মধ্যে সীমাবদ্ধ একটি column যোগ করেছি। এছাড়া content, slug ও status নামের আরও তিনটি column যোগ করেছি। আশা করি এগুলার টাইপটি বুজতে পেরেছেন। আমি এই কলামগুলা তৈরি করার জন্য যে মেথড গুলো ব্যবহার করেছি সেগুলাকে বলে কলাম মেথড(Column methods)।

**আপনাদের সুবিধার্থে নিচে কলাম মেথড গুলোর লিস্টটি লারাভেল ৫.৪ ডকুমেন্টেশন থেকে কপি করে আনলাম।**

Command  | Description
------------- | -------------
`$table->bigIncrements('id');`  |  Incrementing ID (primary key) using a "UNSIGNED BIG INTEGER" equivalent.
`$table->bigInteger('votes');`  |  BIGINT equivalent for the database.
`$table->binary('data');`  |  BLOB equivalent for the database.
`$table->boolean('confirmed');`  |  BOOLEAN equivalent for the database.
`$table->char('name', 4);`  |  CHAR equivalent with a length.
`$table->date('created_at');`  |  DATE equivalent for the database.
`$table->dateTime('created_at');`  |  DATETIME equivalent for the database.
`$table->dateTimeTz('created_at');`  |  DATETIME (with timezone) equivalent for the database.
`$table->decimal('amount', 5, 2);`  |  DECIMAL equivalent with a precision and scale.
`$table->double('column', 15, 8);`  |  DOUBLE equivalent with precision, 15 digits in total and 8 after the decimal point.
`$table->enum('choices', ['foo', 'bar']);` | ENUM equivalent for the database.
`$table->float('amount', 8, 2);`  |  FLOAT equivalent for the database, 8 digits in total and 2 after the decimal point.
`$table->increments('id');`  |  Incrementing ID (primary key) using a "UNSIGNED INTEGER" equivalent.
`$table->integer('votes');`  |  INTEGER equivalent for the database.
`$table->ipAddress('visitor');`  |  IP address equivalent for the database.
`$table->json('options');`  |  JSON equivalent for the database.
`$table->jsonb('options');`  |  JSONB equivalent for the database.
`$table->longText('description');`  |  LONGTEXT equivalent for the database.
`$table->macAddress('device');`  |  MAC address equivalent for the database.
`$table->mediumIncrements('id');`  |  Incrementing ID (primary key) using a "UNSIGNED MEDIUM INTEGER" equivalent.
`$table->mediumInteger('numbers');`  |  MEDIUMINT equivalent for the database.
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent for the database.
`$table->morphs('taggable');`  |  Adds unsigned INTEGER `taggable_id` and STRING `taggable_type`.
`$table->nullableMorphs('taggable');`  |  Nullable versions of the `morphs()` columns.
`$table->nullableTimestamps();`  |  Nullable versions of the `timestamps()` columns.
`$table->rememberToken();`  |  Adds `remember_token` as VARCHAR(100) NULL.
`$table->smallIncrements('id');`  |  Incrementing ID (primary key) using a "UNSIGNED SMALL INTEGER" equivalent.
`$table->smallInteger('votes');`  |  SMALLINT equivalent for the database.
`$table->softDeletes();`  |  Adds nullable `deleted_at` column for soft deletes.
`$table->string('email');`  |  VARCHAR equivalent column.
`$table->string('name', 100);`  |  VARCHAR equivalent with a length.
`$table->text('description');`  |  TEXT equivalent for the database.
`$table->time('sunrise');`  |  TIME equivalent for the database.
`$table->timeTz('sunrise');`  |  TIME (with timezone) equivalent for the database.
`$table->tinyInteger('numbers');`  |  TINYINT equivalent for the database.
`$table->timestamp('added_on');`  |  TIMESTAMP equivalent for the database.
`$table->timestampTz('added_on');`  |  TIMESTAMP (with timezone) equivalent for the database.
`$table->timestamps();`  |  Adds nullable `created_at` and `updated_at` columns.
`$table->timestampsTz();`  |  Adds nullable `created_at` and `updated_at` (with timezone) columns.
`$table->unsignedBigInteger('votes');`  |  Unsigned BIGINT equivalent for the database.
`$table->unsignedInteger('votes');`  |  Unsigned INT equivalent for the database.
`$table->unsignedMediumInteger('votes');`  |  Unsigned MEDIUMINT equivalent for the database.
`$table->unsignedSmallInteger('votes');`  |  Unsigned SMALLINT equivalent for the database.
`$table->unsignedTinyInteger('votes');`  |  Unsigned TINYINT equivalent for the database.
`$table->uuid('id');`  |  UUID equivalent for the database.

**আসুন দেরি না করে আমাদের বানানো এবং আগে থাকা(যদি থাকে) মাইগ্রেশনগুলোকে দিয়ে ডাটাবেজে টেবিলগুলি বানিয়ে ফেলি।**
Terminal এ নিচের কমান্ডটি রান করাইঃ
```bash
php artisan migrate
```
এবং বলছে যে মাইগ্রেশন হয়ে গেছে।
দেরি না করে আপনার ডাটাবেজটি দেখুন। আপনার মাইগ্রেশনগুলোর টেবিল ছাড়াও আরও একটি টেবিল আছে যার নাম migrations. এটি লারাভেল নিজে ব্যবহার করে।

####সর্বশেষ মাইগ্রেশনটি undo করতেঃ
```bash
php artisan migrate:rollback
```
আপনার কোনও মাইগ্রেশনে যদি রিনেম অথবা ড্রপ থাকে তাহলে উপরের command টি error দিবে। এই জন্য, doctrine/dbal নামক একটি প্যাকেজ আপনার প্রোজেক্ট এ যোগ করতে হবে। আমরা কিন্তু [ব্লেড টেমপ্লেটিং](http://laravel.howtocode.com.bd/blade-template.html) অধ্যায়ে নতুন প্যাকেজ যোগ করেছিলাম।
এই প্যাকেজটি দুই ভাবে যোগ করতে পারেন

**প্রথমতঃ**
Terminal এ শুধু নিচের কমান্ডটি রান করুনঃ
```bash
composer require doctrine/dbal
```

**দ্বিতীয়তঃ**
composer.json ফাইলটি খুলুন ও require অংশের শেষে ```"doctrine/dbal": "^2.5"``` যোগ করুন। তাহলে অনেকটা নিচের মতো দেখাবেঃ
```json
"require": {
        "php": ">=5.5.9",
        "laravel/framework": "5.2.*",
        "laravelcollective/html": "5.2.*",
        "doctrine/dbal": "^2.5"
    },
```
এবং নিচের কমান্ডটি রান করুনঃ
```bash
composer update
```
এটা আমাদের সব প্যাকেজ গুলোকে update, প্রয়োজনে add করে নিবে।

**কখন আমাদের rollback করার প্রয়োজন হতে পারে?**

যেকোনো সময়। কিন্তু আসুন একটি পরিস্থিতি এর কথা চিন্তা করি, মনে করি আমারা এই উপরের মাইগ্রেশনটাই মাইগ্রেট করার পর মনে হলোঃ আহারে একটি column নামের বানান ভুল হয়েছে বা আরও একটি ফিল্ড লাগতো তখন আবার নতুন মাইগ্রেশন না লিখে rollback করি, মাইগ্রেশনটাকে প্রয়োজন মতো লিখে নেই এবং আবার মাইগ্রেট কমান্ড রান করাই। কিন্তু প্রোজেক্টটি যদি লাইভ সার্ভারে থাকে তখন হয়ত আমরা কাজটি অন্য ভাবে করবো।

####সব টেবিল গুলো মুছে ফেলতে
কাজের সময় এমন হয় যে অনেক আজে বাজে ডাটা ডাটাবেজ ভরে ফেলে বা অন্য কোনও কারণেও মনে হতে পারে সব টেবিল গুলি মুছে ফেলি। তখন নিচের কমান্ডটি রান করাই

```bash
php artisan migrate:reset
```
এখন পুরা ডাটাবেজ ক্লিয়ার হয়ে যাবে এবং আবার যদি মাইগ্রেট কমান্ড রান করাই তাহলে একে বারে ফ্রেশ একটি ডাটাবেজ পাবো।

####একবারে সব টেবিল মুছে ফেলে নতুন করে তৈরি করতে
এই কাজটিই তো আমরা একটু আগে দুই ধাপে করলাম!! হ্যাঁ, আসুন একটি কমান্ডেই কাজটি সেরে ফেলি।

```bash
php artisan migrate:refresh
```
কমান্ডটি প্রথমে সব মুছে ফেলে আবার তৈরি করে দিবে - ডাটাবেজটি refresh হয়ে যাবে।

আজকে এই পর্যন্তই। এর পরের চ্যাপ্টারে মডেল নিয়ে আলোচনা করা হবে।
