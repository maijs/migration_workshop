# Composer template for migration workshop Drupal site (DrupalCampBaltics 2016)

This project template should provide a kickstart for managing your site
dependencies with [Composer](https://getcomposer.org/).

If you want to know how to use it as replacement for
[Drush Make](https://github.com/drush-ops/drush/blob/master/docs/make.md) visit
the [Documentation on drupal.org](https://www.drupal.org/node/2471553).

## Usage

First you need to [install composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

Secondly you need to [install drush](http://docs.drush.org/en/master/install/). If you don't want to install `drush` globally, you can use the one provided by this template. 

To fetch the code, run the following command in your docroot of your webserver:

```
composer create-project laurisigaunis/migration_workshop:8.0.0 migration_workshop --stability dev --no-interaction --repository https://raw.githubusercontent.com/laurisigaunis/migration_workshop/master/packages.json
```

With `composer require ...` you can download new dependencies to your 
installation.

```
cd migration_workshop
composer require drupal/devel:~1.0
```

After all the dependencies have been fetched, point your browser to http://[your-domain]/migration_workshop/web and install Drupal using the `Standard` installation profile. You can use Mysql or Sqlite storage for the website.

## What does the template do?

When installing the given `composer.json` some tasks are taken care of:

* Drupal will be installed in the `web`-directory.
* Autoloader is implemented to use the generated composer autoloader in `vendor/autoload.php`,
  instead of the one provided by Drupal (`web/vendor/autoload.php`).
* Modules (packages of type `drupal-module`) will be placed in `web/modules/contrib/`
  * This setup already includes the following modules:
    * `migrate_plus` - provides `Url` migration source plugin which is used for JSON and XML migration;
    * `migrate_tools` - provides migration related drush commands;
    * `migrate_source_csv` - provides `CSV` migration source plugin which is used for CSV migration;
    * `migrate_source_example` (a Github project) - contains migration examples.
* Theme (packages of type `drupal-theme`) will be placed in `web/themes/contrib/`
* Profiles (packages of type `drupal-profile`) will be placed in `web/profiles/contrib/`
* Creates default writable versions of `settings.php` and `services.yml`.
* Creates `sites/default/files`-directory.
* Latest version of drush is installed locally for use at `vendor/bin/drush`.
* Latest version of DrupalConsole is installed locally for use at `vendor/bin/drupal`.

## The goal

This workshop is intended to show how migrations can be performed using different methods and sources. This will include migration from CVS and XML files, remote JSON resources and database tables.

## General information about the setup

Every example module depends on `migrate_source_example_setup` module whose purpose is to set-up the data structures into which all migrations will be imported in. The data structures are:
  * `book` content type - for books;
  * `authors` taxonomy vocabulary - for book authors;
  * `categories` taxonomy vocabulary - for book categories;
  * `administrator` and `editor` user roles - for user roles;
  * fields for said content entities which contain source data.

Every example module also depends on `migrate_plus` and `migrate_tools` modules which provide migration related drush commands, migration groups and source plugins for JSON and XML data.

### CSV migration

CSV migration provides an example how migrations can be used if content is stored in multiple CSV files. This migration also includes translations for certain content (book names translated into Latvian) where translations are stored in a separate CSV and migrated independently from the source content (which is in English).

Steps to proceed:

* Enable `migrate_source_example_csv` module by running `drush en migrate_source_example_csv -y` or in UI at `/admin/modules`.
* The /source directory contains the following CSV files:
  * `author.csv` contains book authors;
  * `books.csv` contains book data;
  * `books_translation.csv` contains translations of books (currently in Latvian);
  * `category.csv` contains book genre;
  * `image.csv` contains book image filenames;
  * `user.csv` contains user data.
* Run `drush ms` to confirm that CSV migrations are registered and listed.
* There are multiple ways to run migrations:
  * one by one - `drush mi migrate_source_example_csv_author` - this will migrate only book authors;
  * all group - `drush mi --group=migrate_source_example_csv` - this will migrate all migrations in CSV migration group; 
  * every migration - `drush mi --all`
* After migration manually confirm that content has been imported. Check the following pages:
  * content and translations - `/admin/content`;
  * authors and categories - `admin/structure/taxonomy/manage/author/overview` and `admin/structure/taxonomy/manage/category/overview`;
  * users and their roles - `/admin/people`.
* Run `drush ms` to see how many items are in total, imported or unprocessed.
* Should you decide to undo the migration, run `drush mr --group=migrate_source_example_csv` to rollback the migrations in the group or `drush mr --all` to remove all migrated content and restore the state before the migrations have been run.
* Questions?
* If after content migration new rows are added to the source files, they can be migrated with the same drush commands.

### DB migration

The most common way to migrate is from a legacy database. Example migration from a legacy database contains the same set of books, authors, genres and users as all other migrations.

Steps to proceed:

* Enable `migrate_source_example_db` module by running `drush en migrate_source_example_db -y` or in UI at `/admin/modules`.
* The module will create custom tables in Drupal website database, which for the sake of example will be considered a “legacy” database. The following tables will be created:
  * `migrate_source_example_db_attributes`
  * `migrate_source_example_db_books`
  * `migrate_source_example_db_files`
  * `migrate_source_example_db_users`
* Run `drush ms` to confirm that DB migrations are registered and listed.
* Run the migrations:
  * `drush mi migrate_source_example_db_author`
  * `drush mi --group=migrate_source_example_db`
  * `drush mi --all`
* After migration manually confirm that content has been imported.
* Run `drush mr --group=migrate_source_example_db` to rollback the migrations in the group or `drush mr --all` to remove all migrated content.

### JSON migration

Migration from a JSON resource is another commonly used method to import content into a Drupal site. The content can be served via a REST API or other URL which provides a valid JSON syntax. The data fetcher and parser for JSON resource is provided by `migrate_plus` module, hence it's defined as a dependency.

JSON migration example is structured in a way where content source is considered to be a URL, not a file. The module provides URLs which return a JSON string containing books, book attributes, book images and system users.

Steps to proceed:

* The URLs are the following ([BasePath] is the domain name and base path of your Drupal installation):
  * `[BASE_PATH]/json/books` - contains book data;
  * `[BASE_PATH]/json/attributes` - contains book author and genre;
  * `[BASE_PATH]/json/images` - contains book images;
  * `[BASE_PATH]/json/users` - contains system users.
* Enable `migrate_source_example_json` module by running `drush en migrate_source_example_json -y` or in UI at `/admin/modules`.
* Given that domain name and base path is essential to finding the right location to the content source, the `--uri` parameter needs to be passed to any migration related drush command.
* If your Drupal site is served from http://migration_workshop.dev/web, you drush command for migration status is `drush ms --uri=http://migration_workshop.dev/web`. Use `--uri` parameter for all JSON migration drush commands, otherwise JSON migration operations will not work properly.
* Run `drush ms --uri=[BASE_PATH]` to confirm that JSON migrations are registered and listed.
* Run the migrations:
  * `drush mi migrate_source_example_json_author --uri=[BASE_PATH]`
  * `drush mi --group=migrate_source_example_json --uri=[BASE_PATH]`
  * `drush mi --all --uri=[BASE_PATH]`
* After migration manually confirm that content has been imported.
* Run `drush mr --group=migrate_source_example_json --uri=[BASE_PATH]` to rollback the migrations in the group or `drush mr --all --uri=[BASE_PATH]` to remove all migrated content.

### XML migration

Similarly to JSON migration, the data fetcher and parser for XML files is provided by `migrate_plus` module, hence it's defined as a dependency. While source content in XML format can also be retrieved from a remote resource (URL), for the sake of example XML files in XML example migration module are considered to be a local resource.

Steps to proceed:

* Enable `migrate_source_example_xml` module by running `drush en migrate_source_example_xml -y` or in UI at `/admin/modules`.
* The files are located in /source folder of the module and are the following:
  * attributes.xml - contains book author and genre;
  * books.xml - contains book data and book images;
  * users.xml - contains system users.
* Run `drush ms` to confirm that XML migrations are registered and listed.
* Run the migrations:
  * `drush mi migrate_source_example_xml_author`
  * `drush mi --group=migrate_source_example_xml`
  * `drush mi --all`
* After migration manually confirm that content has been imported.
* Run `drush mr --group=migrate_source_example_xml` to rollback the migrations in the group or `drush mr --all` to remove all migrated content.

### Challenge!

If you are interested in making the migration example module more comprehensive, don't hesitate to create a pull request (https://github.com/wunderkraut/migrate_source_example). The following items are sought for:

* Tests that cover the completion of migrations;
* Advanced usage of process plugins;
* Other migration sources.
