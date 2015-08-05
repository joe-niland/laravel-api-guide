Laravel and PHP Best Practices
===============

Source Control
--------------

1. Only store custom applicaton code in git. 

   Now that we are using composer, all dependencies only need to be referenced in the composer.json file. No need to store them locally.

   Source: [Composer best practices](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md)

2. Similarly, don't store operational directories such as `logs/` in git. These can be created during deployment.

   Use the .gitignore file to implement the above.

Code Generation
----------------

1. Use generators instead of creating code manually. 

   This enforces a consistent approach and will save you a lot of time!

Unit Tests
------------

1. All controller classes must have a corresponding unit test class. Each controller method must have at least one test.

Database Changes
-----------------

1. Create a new migration for each database change. This allows you to track database changes against versions of the app.

  Migrations can cause data loss if care is not taken. Before running in Production, test each migration on your own development site, as well as on a Staging site. 

  The first place to test the impact of a migration is via unit tests.

  When testing on the Staging site, clone the Production data set into the Staging database to make sure that you are testing on a realistic data set.

  It may be necessary to add more logic to a migration to ensure that data loss is avoided.

User Input
------------

1. Never trust or directly use user input. For example, inserting the value from a user's form post directly into the database. All values that are either stored, output, or used otherwise must be validated before taking action on the value.
2. Similarly, never manually generate SQL queries by concatenating hard-coded SQL with user input. This is likely to allow SQL injection attacks.

Code Organisation and Structure
--------------------------------

1. Implement the DRY principle: **D**on't **R**epeat **Y**ourself. Any time you see a given piece of logic appear twice, implement it in a base class or helper method. Having the logic in one location makes it much easier to change this logic in future, rather than having to find every place it is implemented.
    
    In Laravel, you can implement generic helper functions in a reusable way by defining them in classes within `app/lib/` or `app/helpers/`. Once you have done this add these classes' paths to `composer.json` in the "autoload" section, e.g.:

    ```
    "autoload": {
      "classmap": [
          "database"
      ],
      "files": [
          "app/lib/http-helpers.php"
      ]
    },
    ```

    You can then call them from anywhere in your application, therefore this approach should be used for functions that will be used a lot throughout your app.

    This type of global function is not the ideal method because it is not clear exactly how to use the functions, or where they are used.

    In Laravel it is better to implement common functions and behaviours in:
    * event handlers, which can be triggered before/after a route, controller method, etc
    * base classes, for example implementing all desired custom controller behaviour into an `MyDevShopBaseController`
    * [Facades](http://laravel.com/docs/facades)
    * service providers, which allow you to include components into your application in a modular way. They can be managed separately to your web applications, and can be easily included when needed.

2. Create base classes for all model, controller, view classes. Even if you don't end up putting anything in the base class, this approach has the following benefits:
	* Easier to add future requirements, e.g. we need to convert all out controllers so that they allow soft-deletion only. Adding this to the controller base class is a lot less dev/test work that converting each of our controller classes one by one.
	* Encapsulate common functionality in one place (DRY)

Use of External Libraries
---------------------------

1. Always include external libraries via composer. This allow you to easily keep track of which version you are using, and easily update your dependencies when they change.

2. Never change code in an external library. This will break your upgrade path! If you need to vary the behaviour, either:
  + fork the library, fix the issue, and submit a pull request, or
  + derive a sub class from the library's class and override the desired behaviour

Messages
---------------

1. Never include messages directly at the point at which they are used. Instead include them in a resources file, named clearly. Benefits:
   - When you need to change them you only have to look in one file
   - The file can be replaced depending on the user's language settings, thereby translating your application's messages.

   In Laravel, the best place to put messages in the language files. For example: `app/lang/en/notification.php`
