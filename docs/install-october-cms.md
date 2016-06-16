---

template:         article
reviewed:         2016-06-16
title:            Install October CMS on fortrabbit
naviTitle:        October CMS
lead:             October is a free, open-source, self-hosted CMS based on the Laravel PHP framework. Learn how to install and use it on fortrabbit.


group:            Install_guides

websiteLink:      https://octobercms.com/?utm_source=fortrabbit
websiteLinkText:  octobercms.com
category:         CMS
image:            october-cms-mark.png

seeAlsoLinks:
    - app
    - install-laravel-5

keywords:
    - cms
    - install
    - laravel

tags:
    - advanced
    - php
    - install

---


## Get ready

We assume you've already created a New App with fortrabbit which has the [MySQL](mysql), [Memcache](memcache) components enabled - optionally also the [Worker](worker) component. You also need a local [October](https://octobercms.com/docs/console/commands#console-install) installation. You can either use an existing one or start from scratch.


## Install

If you have a running local installation you can skip this step. Otherwise continue by executing locally:

```bash
$ cd ~/Projects
$ composer create-project october/october --prefer-dist MyApp dev-master
```

Create a new file `config/dev/database.php` with the following contents:

```php
return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => 'localhost',
            'port'      => '',
            'database'  => 'your-local-database-name',
            'username'  => 'your-local-database-user',
            'password'  => 'your-local-database-password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
    ],
];
```

To create the local database contents finish with:

```bash
$ php artisan october:up --env=dev
```

### Configure database settings

Create a the database configuratin file in `config/prod/database.php` with the following contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => $secrets['MYSQL']['HOST'],
            'port'      => $secrets['MYSQL']['PORT'],
            'database'  => $secrets['MYSQL']['DATABASE'],
            'username'  => $secrets['MYSQL']['USER'],
            'password'  => $secrets['MYSQL']['PASSWORD'],
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
    ],
];
```

<!--

Now create a database tunnel file, which will be needed later in this guide, under `config/tunnel/database.php` with the following contents:

```php
return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => '127.0.0.1',
            'port'      => 13306,
            'database'  => 'your-app-name',
            'username'  => 'your-app-name',
            'password'  => 'your-app-database-password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
    ],
];
```

**Note**: You can get your App's database credentials using the `secrets` command, eg `ssh git@deploy.eu2.frbit.com secrets your-app-name`

**Note**: This file will be later on placed in the `.gitignore` file so it will not be part of your Git history.

-->

### Configure App settings

Create a new file `config/prod/app.php` with the follwing contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'key' => $secrets['CUSTOM']['APP_KEY'],
    'log' => 'errorlog'
];
```

### Configure cache & session settings

If you plan on running your App in a production PHP plan then you need a shared accessible cache, which Memcache is, since those plans run your App on multiple nodes. If you are just tinkering, then you can skip this topic for now.

First make sure the sessions are using your Memcache component by creating the file `config/prod/session.php` with the following content:

```php
return [
    'driver' => 'memcached'
];
```

Now you need to configure the `memcached` driver, which is the used for session and caching alike, by creating `config/prod/cache.php` containing:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

$servers = [
    ['host' => $secrets['MEMCACHE']['HOST1'], 'port' => $secrets['MEMCACHE']['PORT1'], 'weight' => 100]
];
if (isset($servers['MEMCACHE']['HOST2'])) {
    $servers []= ['host' => $secrets['MEMCACHE']['HOST2'], 'port' => $secrets['MEMCACHE']['PORT2'], 'weight' => 100];
}

return [
    'default' => 'memcached',
    'stores' => [
        'memcached' => [
            'driver'  => 'memcached',
            'servers' => $servers
        ]
    ]
];
```

### Configure file system settings

You most likely require a storage, for user uploads or content generated by your editors - or any other runtime data your App creates. Please don't store those files on the local file system as it is [ephemeral](/quirks#toc-ephemeral-storage). Use our [Object Storage Component](/object-storage) for uploads and static files instead. Once you have booked the Component in the Dashboard the credentials will automatically become available via the [App secrets](/secrets).

October comes already with Flysystem support. Flysystem abstracts the filesystem layer nicely and allows you to simply switch out the adapter. Continue by installing the AWS S3 adapter:

```bash
$ composer require league/flysystem-aws-s3-v2
```

When that is done create the file `config/prod/filesystems.php` to configure an "S3 disk":

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'default' => 's3',
    'cloud'   => 's3',
    'disks'   => [
        's3'  => [
            'driver'   => 's3',
            'key'      => $secrets['OBJECT_STORAGE']['KEY'],
            'secret'   => $secrets['OBJECT_STORAGE']['SECRET'],
            'region'   => $secrets['OBJECT_STORAGE']['REGION'],
            'bucket'   => $secrets['OBJECT_STORAGE']['BUCKET'],
            'base_url' => 'https://'. $secrets['OBJECT_STORAGE']['SERVER']
        ]
    ]
];
```

To use the just configured disk in October you need to create `config/prod/cms.php` containing:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'storage' => [
        'uploads' => [
            'disk'   => 's3',
            'folder' => 'uploads',
            'path'   => $secrets['OBJECT_STORAGE']['URL']. '/uploads',
        ],
        'media' => [
            'disk'   => 's3',
            'folder' => 'media',
            'path'   => $secrets['OBJECT_STORAGE']['URL']. '/media',
        ]
    ]
];
```

## Dashboard settings

Go to the fortrabbit [Dashboard](dashboard), navigate to your App > ENV vars and create:

```
APP_ENV=prod
```

Now switch to App > App secrets and add:

```osterei32
APP_KEY=32CharLongRandomString
```

## Initial deploy

If your local app directory is not already Git controlled it's time to make it so:

```bash
$ git init .
```

Once that's done, edit the contents of the `.gitignore` and remove (or comment out) `composer.lock`:

```
/bootstrap/compiled.php
/vendor
composer.phar
.DS_Store
.idea
.env
.env.*.php
.env.php
php_errors.log
nginx-error.log
nginx-access.log
nginx-ssl.access.log
nginx-ssl.error.log
php-errors.log
sftp-config.json
selenium.php
#composer.lock             << commented out

# for netbeans
nbproject
```

Now add everything to Git, make an initial commit, add your App's remote and push:

```bash
$ git add -A
$ git commit -m 'Initial'
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
$ git push -u fortrabbit master
```

**Note**: Your first deploy takes a long time, because all composer packages need to be installed. Subsequent deployments won't need to do that and will be much faster.

## Setup database

Now that your code is pushed and all config files are created the last step is to setup your App's database. You can do so by [executing a remote commands via SSH](/ssh):

```bash
$ ssh your-app@deploy.eu2.frbit.com php artisan october:up
```

**Note**: Using remote SSH execution, you can run all `artisan` commands.

Done

## Tuning

### Install plugin

You should first install the plugin locally, then in your fortrabbit App:

```bash
# install locally
php artisan --env=dev plugin:install Vendor.Name

# install on fortrabbit
ssh your-app@deploy.eu2.frbit.com php artisan plugin:install Vendor.Name
```

Now add all to Git, commit and push:

```bash
$ git add -A
$ git commit -m 'Plugin installed'
$ git push
```

Once that's done login to your October backend and configure your plugin as usual.


### Scheduled jobs

If you plan on using scheduled jobs then you can configure a new Cron Job in the fortrabbit Dashboard > your App > Worker Jobs > Add a new Cron Job and enter:

* **Name:** `scheduler`
* **Command:** `artisan schedule:run`
* **Interval:** `every minute`


## Troubleshooting

### Thumbnails are not visible

October currently does not support rendering thumbnails from the cloud storage. When using the October's Media browser you can double-click on an image which forces October to use the original image (from the Object Storage) and not the (missing) local thumbnail.
