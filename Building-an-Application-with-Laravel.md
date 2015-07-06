Building an Application with Laravel
====================

This section assumes you are created the project as described in [Working with Laravel](Working-With-Laravel.md).

Source Control
---------------
Before you do anything, add the project directory to git, commit and push. This provides a baseline that will track all changes from the base framework.

1. `vagrant ssh`
2. `cd ~/projects/laravel-api`
3. `git init`
4. `git add .`
5. `git commit -m "initial commit - laravel base project"`
6. `git remote add origin https://github.com/YOUR_REPO/laravel-api.git`
7. `git push -u origin master`

Configuration
-------------

In the last section I explained how to create the base Laravel project. The first step in a new project is to set the configuration as required. 

app/config/app.php
=====================

Laravel's configuration is found in app/config/app.php. You will want to change the following:

debug: you can set this to true to enable detailed error logging when required. This setting must ALWAYS be false in Production.
url: set this to the url you'll be using to access the application, e.g. http://laravel-api.phplocal.dev
timezone: set this to the timezone that's most useful to you. It might be local time or server time.

app/config/database.php
=====================

This file is where all the database connection information is stored. You can remove all the database entries that you won't use. In our case, we can remove everything except for mysql and redis.

Laravel Bootstrap
===============

### Environments

In the bootstrap/start.php file you can set up environments. These allow you to vary application settings depending on where you are running. The homestead VM already has an environment added for development. The start.php file will automatically detect when your code is NOT running on the homestead VM and will detect the environment as 'production'.

Further Reading
===============

[Configuration](http://laravel.com/docs/configuration) describes how to work with Laravel's configuration.
[Laravel API](http://laravel.com/api/index.html) includes a reference for all the classes used by Laravel.
[Lavavel Book](http://laravelbook.com/) has some good tutorials on advanced Laravel usage.
[Laracasts](https://laracasts.com/) are screencasts showing a wide variety of Laravel topics.

How Does Laravel Process a Request
------------------------------------
Just like many other frameworks, laravel uses a single php file `index.php` to handle all requests. When index.php receives a request from the web server, the following will happen:

1. bootstrap/start.php runs - this detects the environment
2. Service providers loaded - these provide services to your application
3. app/start/*.php files are loaded - this initialises your application
4. app/routes.php is loaded
5. Application processes request and generates a Response object
6. Web server sends response back to client.

Further Reading
------------------
This document will summarise the process for creating a Laravel application. I recommend reading the [docs here](http://laravel.com/docs/lifecycle) too.

Let's Do Something!
==========

We're now ready to start building something unique. We will build out an endpoint to handle user registration, modification and authentication.

We'll use generators to scaffold out our models, controllers, and views.
This guide goes through the exact steps you need to do to implement this endpoint. As a reference I've included them in [laravel-api-sample](https://github.com/joe-niland/laravel-api-sample)

But first we need to install a couple of packages:

Prerequisites
==============

First we need to add generator support to the project. This is done with a single line in the composer.json file.

1. Open **dev_root**/laravel-api/composer.json
2. Add the following:

   `
   "require-dev": {
      "way/generators": "2.*",
      "barryvdh/laravel-debugbar": "1.*"
   },
   `

   after `"require": {}`

   The generators will make it fast to build out our project. The debugbar is useful during development.

3. Save and close the file
4. From within the homestead VM, in the **dev_root**/laravel-api/ directory, run the command: `composer update --dev`.

   This will download the generators project and install it.

5. The last step is to add the generators as a Laravel Service Provider:

   Open app/config/app.php and add the following to the 'providers' array:

   `
   'Way\Generators\GeneratorsServiceProvider',
   'Barryvdh\Debugbar\ServiceProvider',
   `

Using the Generators
-----------

Laravel ships with a command line tool called **artisan**. Artisan lets you interact with your project in powerful ways. One of these ways is to perform 'scaffolding', which creates new code files based on templates.

To see the commands made available by the above Service Provider, type `php artisan` from **dev_root*/laravel-api/

![php artisan generator commands](http://images.devs-on.net/Image/piyXjG57vBZzBLe5-Region.png)

### Further Reading

[Laravel 4 Generator Docs](https://github.com/JeffreyWay/Laravel-4-Generators)

