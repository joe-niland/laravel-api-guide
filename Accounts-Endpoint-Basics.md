Accounts Endpoint - Basics
====================================

Most APIs need to authenticate their users so we'll start with user accounts. In this section I will show you how to create basic Account functionality.

Create Database Table
---------------------

We're goint to user Laravel's Migrations feature to automate the creation of the MySQL table to house accounts data.

`php artisan make:migration create_accounts_table --fields="email:string, password:string, type:integer, firstName:string, lastName:string, facebookUserId:string, linkedinUserId:string, platform:string, dateCreated:date, dateUpdated:date"`

This creates the folllowing file:

`app/database/migrations/2014_08_29_215145_create_accounts_table.php`

You can remove this table with the command:

`php artisan generate:migration delete_accounts_table`

If you make a mistake and need to remove a column:

`php artisan generate:migration remove_datecreated_from_accounts_table --fields="dateCreated:date"`

You can also add/remove/change columns by editing the migrations PHP file.

### A note on Migrations

Migrations are an excellent way to build up your database. They provide, in code, the full history of your database changes which means you'll always be able to be sure that you're deploying the correct version of your database schema.

Creating Development Test Data
----------

You can use Laraval's seed feature to add sample data to your application. Again we use the generator to do this:

This command uses the "faker" project, so we first need to add it to our dependencies in composer.json:

1. Add to require-dev:

   `"fzaninotto/faker": "v1.4.0"`

2. `composer update --dev`

3. `php artisan make:seed accounts`

This will generate the seed template that you can fill in with data.

4. Edit the file `app/database/seeds/AccountsTableSeeder.php` and make it look like this:

   ```
   <?php

   // Composer: "fzaninotto/faker": "v1.4.0"
   use Faker\Factory as Faker;

   class AccountsTableSeeder extends Seeder {

      public function run()
      {
         $faker = Faker::create();

         foreach(range(1, 10) as $index)
         {
            Account::create([
               'email' => 'testuser' . $index . '@example.com',
               'firstName' => 'User',
               'lastName' => $index,
               'password' => 'password'
            ]);
         }
      }

   }

   ```

5. The last step is to add this seeder class to the `app/database/seeds/DatabaseSeeder.php` file:

   ```
   <?php

   class DatabaseSeeder extends Seeder {

           /**
            * Run the database seeds.
            *
            * @return void
            */
           public function run()
           {
                   Eloquent::unguard();

                   $this->call('AccountsTableSeeder');
           }

   }
   ```

Create Model
--------------
Now we have the database table in our migrations, we can create a basic, empty model.

`php artisan generate:model Account`

### Testing our Migrations

Now that we've set up a table, some seed data, and a model to store it in we need to test it:

1. Create the Accounts table: `php artisan migrate`
2. Add the seed data: `php artisan db:seed`

Create Controller
------------------

First we'll scaffold out a Controller:

`php artisan generate:controller Account`

We will look at the Controller in more detail later.

Create Views
---------------

`php artisan generate:view account.index`

Create Resources using generate:scaffold
----------------
There is a shortcut to the above. Using generate:resource, we can create all the components we need for the endpoint. This is the easiest way to create everything, but you can use the individual commands when you need to change individual components.

`php artisan generate:scaffold account --fields="email:string, password:string, type:integer, firstName:string, lastName:string, facebookUserId:string, linkedinUserId:string, platform:string"`

**Note:** I left out dateCreated and dateUpdated because Laravel model's include this by default.

You can choose to exclude some components if you've already created them.

Adding the route
---------------------
The above scaffold command will give you the route code you need to add to `app/routes.php`

This will be:

`Route::resource('account', 'AccountsController');`

Testing the Account endpoint default code
-------------------------------

If you haven't already, do these steps:
1. Create the Accounts database table: `php artisan migrate`
2. Add the seed data: `php artisan db:seed`

Then:

1. Open your browser and navigate to http://laravel-api.phplocal.dev/account/1
2. At the moment the endpoint doesn't do anything, it just shows a view. So you'll just see the path to the view:

   /home/vagrant/projects/laravel-api2/app/views/accounts/show.blade.php

In the next steps, we'll configure the Account endpoint to function as required.
