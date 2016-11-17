---

template:      article
naviTitle:     Migrate Universal > Professional
title:         Migrate Universal to Professional
reviewed:      2016-11-11
lead:          "The Universal stack is a good starting point. If you truly need to scale: we offer the Professional stack for exactly that purpose. Here is how you migrate an App from Universal to Professional."
group:         platform
stack:         all

---


The goal of this article is to help you migrate your [Universal App](/app-uni) to a [Professional App](/app-pro). On the way, you will learn how to configure and modify your new App, so that it becomes easily scalable in the future. So scalable indeed, that you only need to click a button in the Dashboard to scale resources whenever you need more performance.

For the purposes of this article, we recommend to chose the newly created Professional App in the App chooser on the right side - this way, this article will show you copy-paste-able code examples.


## Preparation

Make sure your code is ready to scale and you are using up-to-date design patterns. Have a look at our [app design article](/app-design-pro) to learn how to structure your code in a way that it can scale for this. Also have a look at our [general migration guide](/migrating).

Before we show you how to migrate your existing Universal App to a Professional App, you first need make sure that the following applies. Mind that it only needs to apply if your App requires it (eg you don't need cache abstraction, if you don't use caches):

* Your framework/CMS supports storage abstraction, so that you use our [Object Storage](object-storage) Component
* Your framework/CMS supports cache abstraction, so that you can use our [Memcache](memcache-pro) Component
* Your framework/CMS does not use live updates (think "WordPress update button"), or you don't use them
* You are familiar with Git
* You are no fiend of the shell

### Check the install guides

We have written install guides for all popular frameworks and CMS. Have a look at our [all articles list](all-articles) and see if your framework/CMS is available as an install guide for the Professional Stack. Read the article to learn how to deal with it. The following examples focus on general steps to migrate.


### Create a new Professional App

First you need to create a new Professional App in the Dashboard. Choose "Create a new App" on the Dashboard Home > choose a professional Stack App, start with the smallest preset, you can add everything else later on. Later on when all is migrated, you can remove the old Universal App, but that's for later.


### Copy settings

Start by aligning all the settings of the new App with those of the old. Here is your check list:

* PHP Settings
* [Environment variables](env-vars), if any
* [App secets](secrets), if any


## Migration

Once the preparation is completed, you can start with copying data.

### Migrate Database

Assuming you are using MySQL, you need to [import and export](mysql#toc-export-amp-import) your database from the old App to the new one.

### Migrate runtime data

If you are having any kind of run-time data, such as user uploads or content editor uploads, then you need to copy them from the old App to the new App. We recommend to use a GUI like [Cyberduck](https://cyberduck.io/) or [Transmit](https://panic.com/transmit/) to download all runtime data of the old App to your local machine and then upload it back to the [Object Storage](/object-storage) of your new App.

#### Changed URLs

Since the runtime data is now stored in the Object Storage Component of your App, all URLs referencing it must change accordingly. To give you a simple example, assuming there was file `image.png` in a folder `uploads` in the old App:

* Old URL: `http://domain.tld/uploads/image.png`
* New URL: `http://{{app-name}}.objects.frb.io/uploads/image.png`

Many frameworks/CMS support a "global" configuration setting, which allows you to change those URLs with a single line of code. If that is not the case with your framework/CMS, then you need to make sure that all URLs are rendered as shown.

### Use environment variables for configuration

Since Professional Apps only support Git deployment, and you don't want your access details to any service stored plain text in a Git version history, we strongly recommend to use [environment variables](env-vars) (or [secrets](secrets)) to store all your access details (eg MySQL access, 3rd party service access and so on).

### Setup Git repo

If the App's code on your local machine is not already under Git version control, then it's now time to make it so. The general approach is to first define all folders and files which are to be excluded in a `.gitignore` file, then initialize your local App folder as a git repository and finally `git push` to your new Professional App.

```bash
$ cd your/local/app-dir
$ git init .
```

If you App is already under Git version control, then you only need to add your new App's git remote. Also double check your ignore file, to make sure all, which should not be there, is not there.

#### Ignore file

Since the excludes, specified in `.gitignore`, are highly depended on your framework/CMS and your custom code, we can give you only a general advise:

* If you are working with composer:
  * NOT exclude the `composer.lock` or `composer.json` file, if you are working with Composer
  * Exclude the `vendor/` folder
* Exclude upload folders
* Exclude log folders
* Exclude folders containing temporary data

#### Connect

Connecting your App means just to add the new App's remote:

```bash
$ git remote add fortrabbit {{app-name}}@deploy.{{region}}.frbit.com
```

**Note**: If you were already using Git with your old App, then you can use any other name than "fortrabbit" when adding the remote. It's just a helper for you, so that you don't always need to enter the full URL of your repo.

#### First push

Now that your local git repo is connected to your new App, you can:

```bash
$ git push -u fortrabbit master
```

## Review

Once the first deployment is done, it's time to review your new App: are any of the configurations from the preparation are missing or wrong? Do you have been to loose or to strict with the contents in your git ignore file? And so on. It should basically work at this point, just not with your custom domains.

## Finish up: Migrate domains

<!-- TODO: write some more on how to this, first delete the domain with the other App, then add it to the new App, in between change the CNAME … : or link to topic in domain article (check with general mifrate article)  -->

Once your new Professional App is working as expected, you can safely migrate all domains from the old Universal App to the new Professional App. Mind that you also keep the same root path, when doing so. 
