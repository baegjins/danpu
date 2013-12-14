Danpu - MySQL dump tool for PHP
=========

[Packagist](https://packagist.org/packages/rah/danpu) | [Downloads](https://github.com/gocom/danpu/releases) | [Issues](https://github.com/gocom/danpu/issues) | [Donate](http://rahforum.biz/donate/danpu)

[![Build Status](https://travis-ci.org/gocom/danpu.png?branch=master)](https://travis-ci.org/gocom/danpu) [![Latest Stable Version](https://poser.pugx.org/rah/danpu/v/stable.png)](https://packagist.org/packages/rah/danpu)

Danpu is a dependency-free, cross-platform, portable PHP library for backing up MySQL databases. It has no hard dependencies, and is fit for restricted environments where security is key and access is limited. Danpu requires nothing more than access to your database, PDO and a directory it can write the backup to. The script is optimized and has low memory-footprint, allowing it to handle even larger databases.

Danpu supports backing up table structures, the data itself, views and triggers. Created dump files can optionally be compressed to save space, and generated SQL output is optimized for compatibility.

Requirements
---------

Minimum:

* PHP 5.3.0 or newer
* MySQL 4.1.0 or newer
* [PDO](http://php.net/pdo)

Recommended, but optional:

* PHP 5.4.0 or newer
* MySQL 5.0.11 or newer
* [zlib](http://www.php.net/manual/en/book.zlib.php)

Backing up views and triggers requires MySQL 5.0.11 or newer.

Install
---------

Using [Composer](http://getcomposer.org):

    $ composer require rah/danpu:2.5.*

Usage
---------

To create a new backup or import one, configure a new ```Dump``` instance and pass it to one of the worker classes. To begin, first make sure you have included Composer's autoloader file in your project:

```php
require './vendor/autoload.php';
```

If you are already using other Composer packages, or a modern Composer-managed framework, this should be taken care of already. If not, merely add the autoloader to your base bootstrap includes. See [Composer's documentation](http://getcomposer.org) for more information.

### Take a backup

Backups can be created with the ```Export``` class. The class exports the database to a SQL file, or throws exceptions on errors. The file is compressed if the target filename ends to a .gz extension.

```php
use Rah\Danpu\Dump;
use Rah\Danpu\Export;

$dump = new Dump;
$dump
    ->file('/path/to/target/dump/file.sql')
    ->dsn('mysql:dbname=database;host=localhost')
    ->user('username')
    ->pass('password')
    ->tmp('/tmp');

new Export($dump);
```

The database is dumped in chunks, one row at the time without buffering huge amount of data to the memory. This makes the script very memory efficient, and can allow Danpu to handle databases of any size, given the system limitations of course. You physically won't be able backup rows that take more memory than can be allocated to PHP, nor write backups if there isn't enough space for the files.

### Import a backup

Danpu can also import its own backups using the Import class. While the importer works with backups it has made, it doesn't accept freely formatted SQL. The importer is pretty strict about formatting, and expects exactly the same format as generated by Danpu. It expects that values in statements are escaped properly, including newlines, queries have to end to a semicolon and statements preferably should not wrap to multiple lines.

To import a backup, create a new instance of the Import class. It uncompresses any .gz files before importing.

```php
new Rah\Danpu\Import($dump);
```

Where the ```$dump``` would be an instance of Rah\Danpu\Dump as in the *Take backup* example.

Troubleshooting
---------

### Running out of memory, backup taking too long

As with any PHP script, Danpu is restricted by the safe limits you have set for PHP. When dealing with larger databases, taking a backup will take longer and require more memory. If you find yourself hitting the maximum execution time or running out of memory, its time to increase the limits enough to let the script to work.

PHP lets you to change these limits with its [configuration options](http://php.net/manual/en/ini.core.php), which can be modified [temporarily during script execution](http://php.net/manual/en/function.ini-set.php), or in global configuration files. The configuration options you will be the most interested in, are [memory_limit](http://www.php.net/manual/en/ini.core.php#ini.memory-limit) and [max_execution_time](http://www.php.net/manual/en/info.configuration.php#ini.max-execution-time), and possibly [ignore_user_abort](http://php.net/manual/en/function.ignore-user-abort.php). You can change these values in your script before running a backup with Danpu:

```php
ini_set('memory_limit', '256M');
set_time_limit(0);
ignore_user_abort(true);
```

This of course requires that you have access to these values and you actually have more memory to give. Keep in mind that PHP may be affected by other limitations, such as your web server.

Changelog
---------

### Version 2.6.0 - upcoming

* Added: Rah\Danpu\BaseInterface.
* Added: Support for filtering tables by prefix.
* Added: Option to get configuration values from the Rah\Danpu\Dump class.
* Added: Generated dump file now ends to linefeed.
* Added: Option to disable foreign and unique key checks.
* Changed: Writes the DSN to the dump header instead of the old database property and host.

### Version 2.5.0 - 2013/05/22

* Fixed: Database table ignoring.
* Fixed: Importer's decompressor.
* Fixed: Table locking doesn't generate errors if the database doesn't have tables.
* Added: Support for triggers and views.
* Added: Importer supports delimiter keyword.
* Changed: Any SQL error is thrown as exception.

### Version 2.4.0 - 2013/05/16

* Fixed: Makes sure creating the temporary file was successful.
* Added: Adds completed on line to the dump file.
* Added: Option to only dump table structures and no rows.

### Version 2.3.3 - 2013/05/15

* Suggest Zlib, but do not require. Zlib isn't required if the file isn't compressed.

### Version 2.3.2 - 2013/05/14

* Fixed: Define Rah\Danpu\Base::$pdo property as protected.

### Version 2.3.1 - 2013/05/14

* Fixed: Can pass instance of Rah\Danpu\Config to a worker. Rah\Base now correctly hints Rah\Danpu\Config instead of Rah\Danpu\Dump.

### Version 2.3.0 - 2013/05/13

* Fixed: Errors in the given examples. The Rah\Danpu\Dump::temp() should be Rah\Danpu\Dump::tmp().
* Added: Rah\Danpu\Config class. Allows creating dump configs by extending.
* Added: Rah\Danpu\Dump now validates the given configuration option names.
* Added: [PHPunit](http://phpunit.de) tests and [Travis](https://travis-ci.org/gocom/danpu).

### Version 2.2.0 - 2013/05/13

* Added: Database connection details are set using a DSN string. Deprecates Rah\Danpu\Dump::$db and Rah\Danpu\Dump::$host.

### Version 2.1.1 - 2013/05/13

* Fixed: Catch PDOExcepiton.

### Version 2.1.0 - 2013/05/13

* Added: Option to ignore database tables.

### Version 2.0.1 - 2013/05/11

* Fixed: Error in the composer.json.

### Version 2.0.0 - 2013/05/11

* Initial release.
