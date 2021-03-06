---
sidebarDepth: 3
---

# DataImport

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/b169a2f09f864cd5b274ce63008f04b9)](https://www.codacy.com/app/laravel-enso/DataImport?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=laravel-enso/DataImport&amp;utm_campaign=Badge_Grade)
[![StyleCI](https://styleci.io/repos/89221336/shield?branch=master)](https://styleci.io/repos/89221336)
[![License](https://poser.pugx.org/laravel-enso/dataimport/license)](https://packagist.org/packages/laravel-enso/dataimport)
[![Total Downloads](https://poser.pugx.org/laravel-enso/dataimport/downloads)](https://packagist.org/packages/laravel-enso/dataimport)
[![Latest Stable Version](https://poser.pugx.org/laravel-enso/dataimport/version)](https://packagist.org/packages/laravel-enso/dataimport)

Incredibly powerful, efficient, unlimited number of rows, 
queues based Excel importer dependency for [Laravel Enso](https://github.com/laravel-enso/Enso).

This package works exclusively within the [Enso](https://github.com/laravel-enso/Enso) ecosystem.

The front end assets that utilize this api are present in the [ui](https://github.com/enso-ui/ui) package.

For live examples and demos, you may visit [laravel-enso.com](https://www.laravel-enso.com)

[![Watch the demo](https://laravel-enso.github.io/dataimport/screenshots/bulma_006_thumb.png)](https://laravel-enso.github.io/dataimport/videos/bulma_demo_01.mp4)

<sup>click on the photo to view a short demo in compatible browsers</sup>

[![Screenshot](https://laravel-enso.github.io/dataimport/screenshots/bulma_007_thumb.png)](https://laravel-enso.github.io/dataimport/screenshots/bulma_007.png)

## Installation

Comes pre-installed in Enso.

## Features

- allows the import of **big** files with the number of rows only limited by the xlsx file format, 
by splitting the data in chunks and handling them on multiple queues
- uses JSON templates to configure `xlsx` file imports into the application, with minimum custom logic
- import types are defined in the package configuration
- each import type can be validated against required columns, sheets, data types and more
- the Laravel validation is utilized for maximum reuse of existing mechanisms while custom validators can be added when necessary
- an example import type is included by default in the package
- uses [Spout](https://github.com/box/spout) for reading the `xlsx` file
- uses Laravel's queueing system and its auto-balancing features for efficient asynchronous, parallel processing
- blocking file structure validation
- non blocking file contents validation 
- content import issues are made available in the rejected rows summary, a downloadable `xlsx` file with the same structure as the import file,
with an extra column (on each sheet) that will describe all the validation errors for each row
- features real time import progress reporting in the UI
- `before` and `after` hooks which are available during the importing process
- comes with an utility ExcelSeeder class, that can be used to seed your tables using data from excel files

## Usage

1. Publish the configuration, example classes and assets:
    * `php artisan vendor:publish --tag=dataimport-config`
    * `php artisan vendor:publish --tag=dataimport-examples`

2. In the library `samples` folder, `vendor/laravel-enso/dataimport/samples`, we have included a couple of 
 sample import files for you to play with. It is strongly recommended that you check out the provided samples before 
 attempting to create your own importers.

### Configuration
The configuration can be found/published in `config/enso/imports.php` and contains:
 - `validations` - string, flag that sets whether import template validations are also executed in production, 
 valid values are `always`/`local`/`yourEnv` | default `local` 
 - `chunkSize` - number, the number of records in a chunk. It should be adjusted for optimum performance on your machine.
 Not that the size can also be given in the import template, thus overriding the global value
 - `splittingQueue` - string, the name of the queue for the chunk splitting job. Can also be set individually, for each import, in its template
 - `queues` - array, the configuration for all the queues used during the import process. Note that it's good practice to have
 more processes for the splitting queue as this is an intensive process, and it needs to keep the other queues 'busy'.
 Obviously, these queues must be set up in the Laravel `queue` configuration file.
 - `timeout` - number, the Laravel job timeout for the splitting and the rejected summary report generation jobs
 - `errorColumn` - string, the name of the error column used to report issues with the import rows 
 (which appears in the rejected summary xlsx file)
 - `notifications` - array, the list of channels used to notify the user
 - `configs` - configuration array, with what's needed to hook the JSON templates to the import package:
     - `label` - the label visible to the user in the interface
     - `template` - the relative path to the JSON import templates

#### JSON Template structure:
- `timeout`, local overriding configuration for the `enso.dataimport.timeout` option | default is `60 * 4` | optional
- `sheets`, array of sheet configuration objects | required

Note that the importer expects to find just the sheets given in the template, 
meaning it will report an error if there are missing sheets but also if there are extra sheets. 

#### Sheet Configuration object structure:
- `name`, the name of the sheet | required
    - should be lower snake cased if the sheet name contains spaces, so use `sale_entries` instead of `Sale entries`
- `importerClass`, the fully qualified importer class name | required. Here you write the import logic.
- `validatorClass`, the fully qualified custom validator class name, if you are using custom validators | optional
- `chunkSize`, the size of the chunk used during splitting | default is 1000 | optional    
- `columns`, array of column configuration objects | required 

#### Column Configuration object structure:
- `name`, the name of the column | required
    - similar to the sheet name, column names should be lower snake cased, so use `mobile_phone` instead of `Mobile phone`
- `validations`, the desired Laravel (Request) validation that you want applied for this column


Please note that the import does not continue if *structure* errors are encountered, such as missing sheets or columns.
If there are no structure errors and *content* errors are found, 
the rows with errors are skipped and valid rows are imported. 

### Inside the importer class
The importer class given in the JSON template is responsible for doing the actual importing,
 once the file has been validated.

Your importer class needs to implement the `LaravelEnso\DataImport\app\Contracts\Importable` interface 
and implement the `run()` method.

The `run` method receives a row object, which you can use to implement your import logic. 

If you need pre/post import logic for your import, you can then also implement the following interfaces:
- `LaravelEnso\DataImport\app\Contracts\AfterHook` - requires that you implement the 'after' method 
- `LaravelEnso\DataImport\app\Contracts\BeforeHook`- requires that you implement the 'before' method

Inside the two methods you can add your extra logic. 

### Inside the custom validator class
There might be cases where it's not enough to use the Laravel Request Validation methods. For more complex scenarios you can
create a custom validator class and declare it your template.
 
The validator class must extend the `LaravelEnso\DataImport\app\Classes\Validators\Validator` abstract class and implement
the `run()` method.

Here you'll have access to the parent class' `addError(string $error)` method. 
You may use it to add any required issues for the data that fails your custom validation logic.

### The rejected rows summary file

You will notice that the summary file has an identical structure with the import file, with the exception
of one last column, the errors column, that is used to present the issues for each row.

The idea behind creating this kind of report file is for the user to be able to get a list of import issues
in a format that allows him to quickly correct any errors in the file, 
delete the errors column and then simply re-import the summary file.

### Excel Seeder
When seeding your database, you may use regular seeders and fill your tables with random data (as configured) 
but when you have specific data, you may want to set the actual data from the beginning.
While there's more than one way to achieve this, the Excel Seeder helper allows you to seed your table 
by using data provided in an excel file, through an import 
- basically it's a seeder adapter for a regular data import.

There are multiple advantages to this:
- the data is present in an excel file, and can be added and updated by almost any person
- even after initially seeding the database, since the import is going to remain available, additional files can be later imported for updates  

#### Steps
1. create a data import, following your usual flow
2. create your excel seeder class, that will be using the import you added at step 1
    * save it on the `database/seeds/` path
    * note that your class must extend the `ExcelSeeder` class
    * inside, add the `$type` parameter and use for value the type of the import created at the first step
3. add the actual excel file containing the data to be used when seeding
    * the file should be placed on the `storage/app/files` path
    * the name of the file needs to be the type of the import, and and must have an `xlsx` extension
    * since you're going to be committing the file to the repository, don't forget to add it to the `.gitignore` exclusion list
4. run the seed process with `php artisan db:seed` 
    * the seeder will run the import process & the data will be available in the database 
    * you'll also be able to see the result of the import in the data imports index page

## Publishes

- `php artisan vendor:publish --tag=dataimport-config` - configuration files
- `php artisan vendor:publish --tag=dataimport-examples` - example import
- `php artisan vendor:publish --tag=dataimport-mail` - the email templates
- `php artisan vendor:publish --tag=enso-config` - a common alias for when wanting to update configuration,
once a newer version is released, can be used with the `--force` flag
- `php artisan vendor:publish --tag=enso-mail` - a common alias for when wanting to update the email templates,
once a newer version is released, can be used with the `--force` flag

## External dependencies

- [Spout](https://github.com/box/spout)

## Contributions

are welcome. Pull requests are great, but issues are good too.

## License

This package is released under the MIT license.
