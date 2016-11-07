---

template:         article
reviewed:         2016-09-30
title:            Install October CMS on fortrabbit
naviTitle:        October CMS
lead:             October is a free, open-source, self-hosted CMS based on the Laravel PHP framework. Learn how to install and use it on fortrabbit.
group:            Install_guides
stack:            uni
proLink:          install-october-cms-pro

websiteLink:      https://octobercms.com/?utm_source=fortrabbit
websiteLinkText:  octobercms.com
category:         CMS
image:            october-cms-mark.png

keywords:
    - cms
    - install
    - laravel

---

<!-- TODO: Cannot do, web installer still broken -->


## Get ready

We assume you've already created a New App with fortrabbit. You should also have a [PHP development environment](/local-development) running on your local machine. You also need a local [October installation](https://octobercms.com/docs/setup/installation). You can either use an existing one or start from scratch.

### Root path

In the Dashboard: [Set the root path](/app#toc-set-a-custom-root-path) of your App's domains to **public**.

<div markdown="1" data-user="known">
[Change the root path for App URL of the App **{{app-name}}**](https://dashboard.fortrabbit.com/apps/{{app-name}}/domains/{{app-name}}.frb.io/docroot)
</div>

### Environment variables

Go to the Dashboard and add the following [environment variables](/env-vars) to your App:

```osterei32
APP_ENV=prod
APP_KEY=ClickToGenerate
```

<div markdown="1" data-user="known">
[Change ENV vars of the App **{{app-name}}**](https://dashboard.fortrabbit.com/apps/{{app-name}}/vars)
</div>

## Install October CMS with SFTP

The most straight forward way to install October CMS is by downloading it to your local machine and then upload it to your fortrabbit App:

1. Download [the latest](http://octobercms.com/download) archive
2. Unpack the archive locally
3. Open an SFTP client
4. Connect to your fortrabbit App (access credentials)
3. Upload the **contents** of the local `install-master` to the remote `htdocs` folder

### Configure in the browser

Now visit [{{app-name}}.frb.io/install.php](https://{{app-name}}.frb.io/install.php) in the browser and commence with the guided web installation. In the database form enter:

* **Database Type**: MySQL
* **MySQL Host**: `{{app-name}}.mysql.{{region}}.frbit.com`
* **MySQL Port**: leave empty or `3306`
* **Database Name**: `{{app-name}}`
* **MySQL Login**: `{{app-name}}`
* **MySQL Password**: `{{your-database-password}}`




## Install

If you have a running local installation you can skip this step. Otherwise continue by executing locally:

```bash
$ composer create-project october/october --prefer-dist {{app-name}} dev-master
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

<!-- TODO: ENV detection -->


### MySQL

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

### App settings

Create a new file `config/prod/app.php` with the following contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'key' => $secrets['CUSTOM']['APP_KEY'],
    'log' => 'errorlog'
];
```

### Memcache

Cache & session settings: If you plan on running your App in a production PHP plan then you need a shared accessible cache, which [Memcache](/memcache) is, since those plans run your App on multiple Nodes. If you are just tinkering, then you can skip this topic for now.

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

### Object Storage

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

## Initial deploy

If your local app directory is not already [Git controlled](/git) it's time to make it so:

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

Back in the terminal:

```bash
# Add everything to Git
$ git add -A

# Make an initial commit
$ git commit -m 'Initial'

# Add your App's remote Git repo
$ git remote add fortrabbit {{ssh-user}}@deploy.{{region}}.frbit.com:{{app-name}}.git

# Push to deploy
$ git push -u fortrabbit master
```

Your first deploy takes a long time, because all composer packages need to be installed. Subsequent deployments won't need to do that and will be much faster.

## Setup database

Now that your code is pushed and all config files are created the last step is to setup your App's database. You can do so by executing a [SSH remote command](/remote-ssh-execution):

```bash
# Run artisan command via SSH remote execution:
$ ssh {{ssh-user}}@deploy.{{region}}.frbit.com php artisan october:up
```

When it is done you can visit your App URL in the browser:

* [{{app-name}}.frb.io](https://{{app-name}}.frb.io)



- - -

## Tuning

### Install plugin

You should first install the plugin locally, then in your fortrabbit App:

```bash
# install locally
php artisan --env=dev plugin:install Vendor.Name

# install on fortrabbit
ssh {{ssh-user}}@deploy.{{region}}.frbit.com php artisan plugin:install Vendor.Name
```

Now add all to Git, commit and push:

```bash
$ git add -A
$ git commit -m 'Plugin installed'
$ git push
```

Once that's done login to your October backend and configure your plugin as usual.


### Worker

If you plan on using scheduled jobs then you can configure a new Cron Job in the fortrabbit Dashboard > {{app-name}} > Worker > Add a new Cron Job and enter:

* **Name:** `scheduler`
* **Command:** `artisan schedule:run`
* **Interval:** `every minute`

For this, you need to book the [Worker Component](/worker) for your App.

## Troubleshooting

### Thumbnails are not visible

October currently does not support rendering thumbnails from the cloud storage. When using the October's Media browser you can double-click on an image which forces October to use the original image (from the Object Storage) and not the (missing) local thumbnail.
