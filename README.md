# Composer template for migration workshop Drupal site (DrupalCampBaltics 2016)

This project template should provide a kickstart for managing your site
dependencies with [Composer](https://getcomposer.org/).

If you want to know how to use it as replacement for
[Drush Make](https://github.com/drush-ops/drush/blob/master/docs/make.md) visit
the [Documentation on drupal.org](https://www.drupal.org/node/2471553).

## Usage

First you need to [install composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

> Note: The instructions below refer to the [global composer installation](https://getcomposer.org/doc/00-intro.md#globally).
You might need to replace `composer` with `php composer.phar` (or similar)
for your setup.

After that you can create the project:

```
composer create-project laurisigaunis/migration_workshop:8.0.0 migration_workshop --stability dev --no-interaction --repository https://raw.githubusercontent.com/laurisigaunis/migration_workshop/master/packages.json
```

With `composer require ...` you can download new dependencies to your
installation.

```
cd migration_workshop
composer require drupal/devel:~1.0
```

The `composer create-project` command passes ownership of all files to the
project that is created. You should create a new git repository, and commit
all files not excluded by the .gitignore file.

## What does the template do?

When installing the given `composer.json` some tasks are taken care of:

* Drupal will be installed in the `web`-directory.
* Autoloader is implemented to use the generated composer autoloader in `vendor/autoload.php`,
  instead of the one provided by Drupal (`web/vendor/autoload.php`).
* Modules (packages of type `drupal-module`) will be placed in `web/modules/contrib/`
* Theme (packages of type `drupal-theme`) will be placed in `web/themes/contrib/`
* Profiles (packages of type `drupal-profile`) will be placed in `web/profiles/contrib/`
* Creates default writable versions of `settings.php` and `services.yml`.
* Creates `sites/default/files`-directory.
* Latest version of drush is installed locally for use at `vendor/bin/drush`.
* Latest version of DrupalConsole is installed locally for use at `vendor/bin/drupal`.

## FAQ

### How can I apply patches to downloaded modules?

If you need to apply patches (depending on the project being modified, a pull
request is often a better solution), you can do so with the
[composer-patches](https://github.com/cweagans/composer-patches) plugin.

To add a patch to drupal module foobar insert the patches section in the extra
section of composer.json:
```json
"extra": {
    "patches": {
        "drupal/foobar": {
            "Patch description": "URL to patch"
        }
    }
}
```
## Workshop tasks

This workshop is intended to show how migrations can be performed using different methods and sources. This will include migration from CVS and XML files, remote JSON resources and database tables.

### CSV migration

CSV migration provides an example how migrations can be used if content is stored in multiple CSV files. This migration also includes translations for certain content (book names translated into Latvian) where translations are stored in a separate CSV and migrated independently from the source content (which is in English).


Steps:

* Enable `migrate_source_example_csv` module by running `drush en migrate_source_example_csv -y` or in UI at `/admin/modules`. It will enable all necessary modules: `migrate_source_example_csv`, `migrate_tools`, `migrate`, `migrate_plus`, `content_translation`, `language`, `migrate_source_example_setup`, `migrate_source_example`, `migrate_source_csv`. (`Content_translation` and `language` modules are required for multilingual content.)
* Every example module depends on `migrate_source_example_setup` module whose purpose is to set-up the data structures into which all migrations are going to be imported in. The data structures are:
  * `book` content type - for books;
  * `authors` taxonomy vocabulary - for book authors;
  * `categories` taxonomy vocabulary - for book categories;
  * `administrator` and `editor` user roles - for user roles.
* Lets take a look in source files in module:
  * Under /source there is lot of .csv files.
    * `author.csv` contains author of book data
    * `books.csv` contains books and all related info
    * `books_translation.csv` contains translations of books and all related info. Currently in Latvian.
    * `category.csv` contains book genre
    * `image.csv` contains image names
    * `user.csv` contains user data
* Drush commnds for migration - https://www.drupal.org/project/migrate_tools
* Do migration listing with `drush ms`
* Lets migrate things
  * one by one - `drush mi migrate_source_example_csv_author`
  * all group - `druhs mi --group=migrate_source_example_csv`
  * every migration what is available(listed under drush ms)  - `druhs mi --all`
* Check what and where is migrated(users, terms, nodes, translations)
* `drush mr --all` removes all migrations and content will be as it was before.
* Questions?
* Lets say, customer is added new rows to source file for books(or added missing translations for already migrated books) and he or she wants to migrate those as well. There is nothing simpler to just run once again `drush mi --group=migrate_source_example_csv` or `--all`, or specific migration.

### DB migration
Most common way is migrate from database. So at first we will migrate from db. We wil migrate users, terms, files and nodes. This migration includes also translations for nodes.

* By enabling `migrate_source_example_db module`, it saves source in same db where Drupal is installed, but in custom tables.
* Lets take a look in source db tables: `migrate_source_example_db_*`
  * `attributes`
  * `books`
  * `files`
  * `users`
* Do some listing with `drush ms`
* Lets migrate things
  * `drush mi migrate_source_example_db_author`
  * `druhs mi --group=migrate_source_example_db`
  * `druhs mi --all`
* Check what and where is migrated(users, terms, nodes, translations)
* `drush mr --all` removes all migrations and content will be as it was before.

### JSON migration
Other most common source of where migrate from. This could be source from REST API or other valid url. It use URL souce plugin

* Enable `migrate_source_example_json` module.
* Lets tak a look in source files and how can we access them by path:
  * [BasePath]/json/books
  * [BasePath]/json/attributes
  * [BasePath]/json/images
  * [BasePath]/json/users
* Drush commands are basically the same as for cvs and db, but this time we need to add absolute base path.
* For example json book source is under http://migration_workshop.dev/web/json/books, so drush command for status would be `drush ms --uri=migration_workshop.dev/web` Use `--uri` parameter for all json migration drush commands
* `drush mi --all --uri=BasePath`
* `drush mr --all --uri=BasePath`

### XML migration
Last one for this workshop - migration from XML. It use also same URL plugin as JSON migration did.

* Enable `migrate_source_example_xml` module.
* Source files
  * attributes.xml
  * books.xml
  * users.xml
* `drush ms`
* `drush mi --group=migrate_source_example_xml`


