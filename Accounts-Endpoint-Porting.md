Creating the Account endpoint
================================

Most APIs need to authenticate their users so we'll start with user accounts. In this section I will show you how to create basic Account functionality.

Creating the endpoint version
-----------------------------

We need to add v1 to all API URL's. The reason we do this is to future-proof our API. If we ever need to make breaking changes in future releases, we can create a v2 prefix and host the new version of the API there. In this way we won't break legacy integrations.

This is easily accomplished in app/routes.php, by adding a Resource group:

```
Route::group(array('prefix' => 'v1'), function() {
        Route::resource('account', 'AccountsController');
});
```

Implementing Account GET as JSON
------------------------------------

The scaffolding added View creation calls to our controller, but to return JSON we need to change this.

1. Open `**dev_root**/app/controllers/AccountsController.php`
2. Find the show method, and change the folllowing line:

  ```
  return View::make('accounts.show', compact('account'));
  ```

  to:

  ```
  return Response::json($account);
  ```

3. While we're in here, let's also delete the controller methods `create` and `edit`. These display web forms to allow the creation/updating of an account and so for our API we don't need them.

4. That's all that's required to convert our model object to a JSON response. Now we can test it.

5. Open `http://laravel-api.phplocal.dev/v1/account/1` in your browser and you'll see:

  ![Accounts controller response](http://images.devs-on.net/Image/90spMrQT3EugQK0N-Region.png)

**NOTE:** there may be situations where building the JSON response within a view will be required. In this simple case, it's OK to create the response directly in the Controller.

Adding validation
------------------

There are a few ways you can implement validation, but I'm going with executing validation from the controller, using rules defined in the model. For the Account model, I added the following rules, based on https://github.com/phplocalmobile/laravel-api/blob/master/lib/Model_Account.php, with a few additions:

```
<?php

   public static $rules = [
      'firstName' => 'required|max:100',
      'lastName' => 'required|max:100',
      'email' => 'unique:accounts|required|email',
      'password' => 'required|between:8,60',
      'facebookUserId' => 'unique:accounts',
      'linkedinUserId' => 'unique:accounts'
   ];
?>
```

**NOTE:** the max length of password is 60 because it will be stored in a hashed format. See below for details.

See [Laravel Validation docs](http://laravel.com/docs/validation) for full explanation of validation rules.

It's very important to create a validation rule for each value that will be set on a model.

We also need to protect certain fields from mass assignment. This means they cannot be accidentally set when creating/updating a model. They must be set explicitly by accessing the property. In Laravel this is accomplished with the `fillable` and `guarded` model properties:

```
<?php

  // Defines fields that can be set via mass assignment, e.g. Account::create()
  protected $fillable = ['firstName', 'lastName', 'email', 'password', 'facebookUserId', 'linkedinUserId'];

  // Define fields that cannot be set by mass assignment
  protected $guarded = [ 'id', 'created_at', 'updated_at' ];

?>
```

Setting `$fillable` is extremely important because it stops any extra querystring or json parameters being sent to the model save method. So always set `$fillable`.

`$guarded` allows you to explicitly blacklist the ones you don't want set from a request. So this is best used for fields that you know you'll never be setting via a `create` or `update` method on the model.

Fillable will always override guarded, so if you set the same field in both, it _will_ be settable via mass assignment.

Creating Platform as an Enum
-----------------------------

In the above migration, we just created the Account.Platform field as a string field. Rather than manually translating strings into ints, we can use an *enum*.

We'll now create a new migration to make this change:

1. `php artisan migrate:make modify_accounts_table_platform`

2. Edit this file `app/database/migrations/2014_09_04_193941_modify_accounts_table_platform.php`:

  ```
  <?php

  use Illuminate\Database\Schema\Blueprint;
  use Illuminate\Database\Migrations\Migration;

  class ModifyAccountsTablePlatform extends Migration {

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
      // Drop the platform column and recreate it as an enum
      Schema::table('accounts', function($table)
      {
        $table->dropColumn('platform');
      });
      Schema::table('accounts', function($table)
      {
        $table->enum('platform', array('unknown', 'ios', 'android'))->default('unknown');
      });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
      // Replace the enum column with the original string column:
      Schema::table('accounts', function($table)
      {
        $table->dropColumn('platform');
      });
      Schema::table('accounts', function($table)
      {
        $table->string('platform');
      });
    }

  }

  ?>
  ```

3. We now need to modify the AccountsTableSeeder class, to seed the platform:

  ```
  Account::create([
          'email' => 'testuser' . $index . '@example.com',
          'firstName' => 'User',
          'lastName' => $index,
          'password' => 'password',
          'dateCreated' => date("Y-m-d H:i:s"),
          'platform' => array('unknown', 'ios', 'android')[rand(0,2)]
          ]);
  ```

4. And now, we'll test the migration and re-seed:

  ```
  php artisan migrate:refresh && php artisan db:seed
  ```

I also did a similar migration to the above, to add the status column to Account. See: `app/database/migrations/2014_09_05_213930_modify_accounts_table_status.php`

**Note:** instead of using account status deleted, you could use [Soft Deletes](http://laravel.com/docs/eloquent#soft-deleting)

Adding the Company model
---------------------------

Accounts link to Companies, so we need to add a model and set up the references. Company doesn't need an endpoint, so we'll just create a database table, seed data, a model and a migration.

1. Create the database table: `php artisan generate:migration create_companies_table --fields="companyName:string, companyLogo:string, account_id:integer"`

1. Create the database seeder class: `php artisan generate:seed companies`

2. Edit the seeder class. Note we're going to add a reference to the accounts table.
    
  ```
  <?php

   // Composer: "fzaninotto/faker": "v1.4.0"
   use Faker\Factory as Faker;

   class CompaniesTableSeeder extends Seeder {

        public function run()
        {
                $faker = Faker::create();

                foreach(range(1, 10) as $index)
                {
                        // Get a random account
                        $account_id = DB::table('accounts')
                                        ->select('id')
                                        ->where('id', rand(1,10))
                                        ->first()
                                        ->id;

                        Company::create([
                                'account_id' => $account_id,
                                'companyName' => 'testCompany' . $index,
                                'companyLogo' => 'http://lorempixel.com/300/250'
                        ]);
                }
        }

  }
  ```

2. Add this seeder class to the `app/database/seeds/DatabaseSeeder.php` file. It must go **after** Accounts, because Companies depend on accounts.

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
                   $this->call('CompaniesTableSeeder');
           }

   }
   ```

2. Generate a model for Company: `php artisan generate:model Company`. This creates `app/models/Company.php`.

  Notice that the model doesn't contain all the fields we specified in the migration. Laravel's ORM (Eloquent) generates the model properties dynamically.

3. Edit the model file:

  ```
  class Company extends \Eloquent {

    // Company validation
    public static $rules = [
      'companyName' => 'required'
    ];

    // Define fields that cannot be set by mass assignment
    protected $guarded = [ 'id', 'created_at', 'updated_at' ];
  }
  ```

3. Create the Companies table: `php artisan migrate`

4. Add the seed data: `php artisan db:seed`

5. Now we can create the relationship between Account and Company:

  We need to add the foreign key addition/removal to the companies migration file:

  **app/database/migrations/2014_09_02_204912_create_companies_table.php**:

  ```
  class CreateCompaniesTable extends Migration {

        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
                Schema::create('companies', function(Blueprint $table)
                {
                        $table->increments('id');
                        $table->integer('account_id')->unsigned();
                        $table->foreign('account_id')->references('id')->on('accounts');
                        $table->string('companyName');
                        $table->string('companyLogo');
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
                Schema::table('companies', function($table) {
                        $table->dropForeign('companies_account_id_foreign');
                });

                Schema::drop('companies');
        }
  }
  ```

6. Now we need to add the following code to the models to indicate the relationship:

  **app/models/Account.php**:

  ```
  // Relationship to company
  public function company() {
    return $this->hasOne('Company');
  }
  ```

  **app/models/Company.php**:

  ```
  // Relationship to Account
  public function account() {
    return $this->belongsTo('Account');
  }
  ```

  **Note:** By convention, Laravel will assume that the Company table has a foreign key called account_id.

7. And then we modify the Accounts controller to load the company data as well, using the `with` method:

  ```
  /**
   * Display the specified account.
   *
   * @param  int  $id
   * @return Response
   */
  public function show($id)
  {
    $account = Account::with('company')->findOrFail($id);

    return Response::json($account);
  }
  ```

  The `with` method tells the model to load the data linked to by the company relationship.

7. Finally, we create and test the relationship by running the migration and reseeding the tables:

  ```
  php artisan migrate:refresh && php artisan db:seed
  ```

  migrate:refresh removes all migrations and then re-applies them. If you receive any errors, comment out the dropForeign command in the above file, then re-add it after the migrations have been refreshed.

8. Let's check the output at http://laravel-api.phplocal.dev/v1/account/2:

  ![accounts with companies](http://images.devs-on.net/Image/Da6Q0VRQGForAsvI-Region.png)

Accounts POST (Create)
----------------------

The POST HTTP method on the accounts endpoint will create a new account using data passed in the request. Let's look at the default `store` method created on our AccountsController:

```
/**
   * Store a newly created account in storage.
   *
   * @return Response
   */
  public function store()
  {
    $validator = Validator::make($data = Input::all(), Account::$rules);

    if ($validator->fails())
    {
      return Redirect::back()->withErrors($validator)->withInput();
    }

    Account::create($data);

    return Redirect::route('accounts.index');
  }
```

We need to make the following changes:

1. Change the return lines to return JSON data.
2. Encrypt the password before it's stored.
3. Include a notification on account creation success.

We'll ensure the password is encrypted by adding a custom setter to the Account model:

```
  // Password setter - ensure hashing is applied
  public function setPasswordAttribute($clearPassword) {
    $this->attributes['password'] = Hash::make($clearPassword);
  }
```

Here's the modified controller `store` method:

```
  public function store()
  {
    $validator = Validator::make($data = Input::all(), Account::$rules);

    if ($validator->fails())
    {
      return Response::JSON( $validator->errors() );
    }

    $account = Account::create($data);

    // Add notification to response
    $notification = array ( Lang::get('notification.account_create_success') );

    return Response::JSON( array($account, 'notifications' => $notification) );
  }
```

The notification message is loaded from a language file. These are stored in: `app/lang/en/notification.php`. This is added to the response, keyed by the word 'notifications'.

**Note:** I've removed the password from the JSON response since it should generally not be passed back to the client.

As a safeguard we should also specify in the Account model that password should never be returned in the response. We do this by adding the hidden property to the Account model class:

```
  // Fields that should never be returned in a response
  protected $hidden = [
    'password'
  ];
```

Also remember that we set the `guarded` property on the Account model. This stops the id, created_at and updated_at fields being set via the `Account::create` method.

Now we can test adding a new account using curl:

`curl -i -X POST -H "Accept: application/json" --data 'firstName=testuser&lastName=testuserlast&email=testuser@example.com&password=supersecret' laravel-api.phplocal.dev/v1/account`

If we get errors, then it can be easier to use a browser plugin to submit the post, so we can see the error output. Postman is a good choice for Chrome:

![postman API POST](http://images.devs-on.net/Image/3SfZErKq7zAi6Rsz-Region.png)

You can also use the `php artisan tail` command to show you the errors that occur as you submit requests.

Here's the ouput of the curl:

![testing the POST API](http://i.gyazo.com/329e3a42fbe1a601382d33cdf9fa06b0.png)

You can see that the validation failed because the email we tried to use was not unique. The validation rules defined in the model did not allow this to proceed to being stored.

Let's try again with a unique email:

`curl -i -X POST -H "Accept: application/json" --data 'firstName=testuser&lastName=testuserlast&email=testuser.unique@example.com&password=supersecret' laravel-api.phplocal.dev/v1/account`

### Sending an email on Account Create

We also need to email the admin when a new user is created. Laravel support Mailgun out of the box, so we'll set up that support now:

1. Add "guzzlehttp/guzzle": "~4.0" to composer.json
2. composer update
3. Edit `app/config/mail.php` and change driver to 'mailgun'
4. Edit `app/config/services.php` and add mailgun domain and API key:
  ```
   'mailgun' => array(
                'domain' => 'sandboxe361429cb6e04a618d4627e6f80da1a7.mailgun.org',
                'secret' => 'REMOVED',
                ),
  ```
5. Modify the controller store method to send the registration email:
  ```
  public function store()
  {
    $validator = Validator::make($data = Input::all(), Account::$rules);

    if ($validator->fails())
    {
      return Response::JSON( $validator->errors() );
    }

    $account = Account::create($data);

    // Add notification to response
    $notification = array ( Lang::get('notification.account_create_success') );

    // Send registration email
    Mail::send('emails.registration.welcome', $account->toArray(), function($message)
    {
        $message->to($account->email, $account->fullName)->subject('Welcome to Taal!');
    });

    return Response::JSON( array($account, 'notifications' => $notification) );
  }
  ```

  Note, we didn't specify a 'from' address. I've configured this in the mail config at `app/config/mail.php`:

  ```
  'from' => array('address' => 'hello@taal.co', 'name' => 'Taal'),
  ```

  Also, note we've referenced an email view.

6. Create the email view at `app/views/emails/registration/welcome.blade.php`:

  `php artisan generate:view emails.registration.welcome`

7. Since we don't want to send mails in development we'll set `pretend` to true in `app/config/mail.php`. As part of the deployment process, these values will be changed for Production.

Accounts PUT (Update)
----------------------

The `update` method on the `AccountsController` is almost the same as the `store` method. It attempts to find the account by id, then validates the updated data, then saves the changes.

We need to:
1. Modify the returns to return JSON
2. Add the notification message to the response.

**app/controllers/AccountsController.php**:

```
public function update($id)
  {
    $account = Account::findOrFail($id);

    $validator = Validator::make($data = Input::all(), Account::$rules);

    if ($validator->fails())
    {
      return Response::JSON( $validator->errors() );
    }

    $account = $account->update($data);

    // Add notification to response
    $notification = array ( Lang::get('notification.account_update_success') );

    return Response::JSON( array($account, 'notifications' => $notification) );
  }
```

**Note:** at the moment we're using the same validation rules for creation and update. Depending on the needs of your API's client, you may need to vary the rules used for update.

A simple way to do this is to modify the model's `$rules` array to have two sets of rules, e.g.:

```
public static $rules = [
  'create' => [
    'firstName' => 'required',
    'lastName' => 'required',
    'email' => 'unique:accounts|required|email',
    'password' => 'required|min:8',
    'facebookUserId' => 'unique:accounts',
    'linkedinUserId' => 'unique:accounts'
  ],
  'update' => [
    'email' => 'unique:accounts|email'
    'password' => 'min:8'
  ]
  ];
```

With this approach you have to vary the rules used from within the Controller method.

Another way to do it is to use the `required_without` validation command with the `id` field, e.g.

```
public static $rules = [
    'firstName' => 'required_without:id',
    'lastName' => 'required_without:id',
    'email' => 'unique:accounts|required_without:id|email',
    'password' => 'required_without:id|min:8',
    'facebookUserId' => 'unique:accounts',
    'linkedinUserId' => 'unique:accounts'
  ];
```

The above has the effect of only enforcing the required condition when the id field is not present in the validation input, i.e. when we are creating a new account, we don't pass the id.

### Validating uniquely constrained fields on update

The above works fine, but there is one pretty big problem:

By default, Laravel's validation doesn't intelligently handle the case where we are updating a unique field. For example, when we pass the fields for update, the email will be included whether it's been changed or not. Laravel's validation will determine that the email is not unique (because it's the same as what is already in the database!) and will fail the validation.

Laravel has a [way to handle this](http://laravel.com/docs/validation#rule-unique) by allowing you to specify that the unique constraint should be ignored for a given ID. However, when using the static array of rules within the model there is no clean way to pass the ID into the rule upon update.

There are numerous custom solutions to this. The one I have chosen is to define a custom base class for our models which will perform validation automatically on saving, and will customise the validation behaviour. This will also require changes to the Controller methods because we no longer have to orchestrate the validation.

See [Improving Model Validation](Improving-Model-Validation.md) for the explanation.

... continuing with the Accounts update.

Now we need to add the notification for a sucessful account update:

**app/lang/en/notification.php**:

```
return array(
  'account_create_success' => 'Account was created successfully',
  'account_update_success' => 'Account was updated successfully'
  );
```

We can test this using curl:

`curl -i -X PUT -H "Accept: application/json" --data 'firstName=newfirst&lastName=newlast&email=testuser.mod@example.com&password=supersecret2' laravel-api.phplocal.dev/v1/account/1`
