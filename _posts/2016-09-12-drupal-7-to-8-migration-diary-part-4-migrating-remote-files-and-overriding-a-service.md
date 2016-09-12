---
ID: 78
post_title: 'Drupal 7 to 8 Migration Diary (Part 4): migrating remote files and overriding a service'
author: admin65534
post_date: 2016-09-12 06:39:30
post_excerpt: ""
layout: post
permalink: >
  http://stevector.com/2016/09/drupal-7-to-8-migration-diary-part-4-migrating-remote-files-and-overriding-a-service/
published: true
---
After a summer dominated by weddings and work conferences I had a few days of open weekends to make progress on the Drupal 7 to 8 migration. One area I was concerned about from the beginning was the handling of some remote files that are not controlled directly by Drupal.

Nerdologues.com has a number of podcasts and the mp3s themselves live on S3 and are referenced by Drupal 7 using the <a href="https://www.drupal.org/project/remote_stream_wrapper">remote stream wrapper module</a>. So Drupal does not have the ability to upload or move these files. It simply points at them. Recently I heard on <a href="https://www.lullabot.com/podcasts/drupalizeme-podcast/drupal-8-deep-dive-with-andrew-juampy-mateu-and-dave">the Lullabot podcast</a> that Dave Reid had ported remote stream wrapper module to Drupal 8 so I figured it was time to make some progress on the migration.

## Stream wrappers

First some background on <a href="http://php.net/manual/en/class.streamwrapper.php">the concept of stream wrappers</a>. Basically, they are a way to change the implementation details for how files are handled depending on a file's protocol. By default, Drupal files are stored with the prefix `public://`. So a file like `https://www.nerdologues.com/sites/default/files/field_image/podcast/YourStorieslogo.jpg` is not stored with the full URL or even a relative URL like `sites/default/files/field_image/podcast/YourStorieslogo.jpg`. Instead it is stored as `public://field_image/podcast/YourStorieslogo.jpg.` So Drupal core has logic for dealing with most of its files with this public prefix. This abstraction layer even allows for moving all public files to a remote location like S3. But that’s not what I did in the Drupal 7 site. The Drupal 7 site has `public://` files and it has files remote files stored with the prefix `http://` or `https://` that are beyond Drupal's direct control.

## Installing the module

There is not yet a field widget in remote stream wrapper module allowing for adding or editing of remote files. (<a href="https://www.drupal.org/node/2775507">though there is an open issue</a>) That OK for me because my top priority is handling migrated data. I won’t actually need a form to add new podcast episodes individually for months when the Drupal 8 site goes live. So if the module can process remote files as they migrate, that is good enough for me.

## Making the migration work

I already had a working migration of files stored as `public://`. <a href="https://github.com/stevector/nerdologues-d8/blob/b95026c6f62c5696de3b9a4a56694486ec348ea7/web/sites/default/config/migrate_plus.migration.upgrade_d7_file.yml">Looking at the yml file defining this migration</a> I saw a section that referred to the protocol of files being migrated.

```yml
source:
  plugin: d7_file
  scheme:
    - public
```

(BTW, all the links to code segments in this post will use the latest commit hash in the repo rather than simply the master branch. By the time you’re reading this post the master branch may have changed.)

This looked like the relevant place to edit, right? Well, kind of. Just adding “http” to the list of schemes here did not work. This migration uses the destination plugin, “entity:file”, <a href="http://cgit.drupalcode.org/drupal/tree/core/modules/file/src/Plugin/migrate/destination/EntityFile.php?h=8.1.x&id=a734cfae5dba958e3922b9291795b72b50a89f5a#n98">which contains a fair bit of logic for handling local files</a>. I decided that rather than trying to make one migration handle local and remote files I would instead add <a href="https://github.com/stevector/nerdologues-d8/blob/b95026c6f62c5696de3b9a4a56694486ec348ea7/web/sites/default/config/migrate_plus.migration.upgrade_d7_file_remote.yml">a second migration</a> (<a href="https://github.com/stevector/nerdologues-d8/blob/b95026c6f62c5696de3b9a4a56694486ec348ea7/web/modules/custom/nerdcustom/src/Plugin/migrate/destination/EntityFileRemote.php">using a custom destination plugin</a>) for importing source files with the `http` and `https` prefix.

## Optimizing the migration

At this point I had two working migrations. I could run the migration for remote files (`drush mi upgrade_d7_file_remote`) and get successful imports. The key detail I wanted to see for the successful imports was the filesize getting processed in Drupal 8. The RSS feed Drupal produces for the actual podcast needs to know the size of each file.

Remote stream wrapper module gets the filesize by making HTTP HEAD requests to the remote server. And that works well when dealing with a single file. It is not great for importing hundreds of files; especially when the migration runs uniquely for every single push I make to GitHub. And it is unnecessary when the source database knows the size of each file.

## Overriding a service

Remote stream wrapper module does its HTTP request inside of a PHP service class. Overridable services are one of the best new features of Drupal 8. It is cleaner than ever to alter the behavior of a Drupal site.<a href="https://github.com/stevector/nerdologues-d8/blob/5c81772d9ba769528c32ba71f855f3b8cf034e25/web/modules/custom/nerdcustom/src/StreamWrapper/CachedHttpStreamWrapper.php"> In this case I’m overriding the service that gets remote filesize to add a querying of the migration source database</a>.

```
      $files = [];
      $results = Database::getConnection('default', 'drupal_7')
        ->select('file_managed')
        ->fields('file_managed')->execute();

       foreach ($results as $result) {
         $files[$result->uri] = [
           'size' => $result->filesize,
           'mtime' => $result->timestamp
         ];
      }
    }
```

I say “cleaner,” but not exactly “clean” in that this code this far from being contrib ready. But it gets the job. To register the overriding service, <a href="https://www.drupal.org/node/2026959">a module needs to implement ServiceModifierInterface (usually by extending ServiceProviderBase)</a>. <a href="https://github.com/stevector/nerdologues-d8/blob/5c81772d9ba769528c32ba71f855f3b8cf034e25/web/modules/custom/nerdcustom/src/NerdcustomServiceProvider.php">The altering is then as simple as</a>

```
  public function alter(ContainerBuilder $container) {
    $definition = $container->getDefinition('stream_wrapper.http');
    $definition->setClass('\Drupal\nerdcustom\StreamWrapper\CachedHttpStreamWrapper');
    $definition = $container->getDefinition('stream_wrapper.https');
    $definition->setClass('\Drupal\nerdcustom\StreamWrapper\CachedHttpStreamWrapper');
  }
```

## Cleanup

Once the site is live and the migration is no longer being run, this override can be removed. I’ll be sad to see it go. I’ve been hearing about the marvels of services for years as Drupal 8 was developed. I had never replaced a service for a real site. I’m glad it worked as well as promised.