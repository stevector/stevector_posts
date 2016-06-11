---
post_title: 'Drupal 7 to Drupal 8 Migration Diary (Part 2): Using Config Installer to delay a canonical database'
post_excerpt: ""
layout: post
published: false
---

*This post is the second in a series about migrating [nerdologues.com](https://www.nerdologues.com/) from Drupal 7 to Drupal 8. [Read the first post here](http://stevector.com/2016/06/drupal-7-to-drupal-8-migration-diary-part-1-what-have-i-gotten-myself-into/).*

## The configuration workflow I want to use

When I started on this project to migrate [nerdologues.com](https://www.nerdologues.com/) from Drupal 7 to Drupal 8 I was excited to use [Drupal 8's Configuration Manager](https://www.drupal.org/documentation/administer/config). My imagined workflow was something like:

* Install Drupal 8 with the core's Standard install profile on my local machine or in a Pantheon Multidev environment.
* Make configuration changes.
* Export configuration changes to code (wooo!) to the normal directory: `sites/default/config`.
* Push committed configuration to GitHub, make pull request.
* CircleCI then installs with the Standard profile, imports configuration from `sites/default/config`, runs tests.

But there is a problem with this workflow. When attempting to do the import of pre-existing configuration (with `drush config-import`) Drupal will complain with a message that includes:

```
The import failed due for the following reasons:                         
Site UUID in source storage does not match the target storage.
```

As far as Drupal core is concerned, `sites/default/config` is meant for configuration from the same site (or at least different copies of the same site). Doing a fresh install of Drupal core means that the pre-existing config in `sites/default/config` is seen as foreign.

## Enter Configuration Installer

[Configuration Installer](https://www.drupal.org/project/config_installer) allows me to use the workflow that I want. (Thanks to [Alex Pott](https://www.drupal.org/u/alexpott) for writing it!) Drupal bases new installations on installation profiles. In Drupal 8, each install profile is expected to provide the configuration it needs in the form of `.yml` files. For most every install profile, those `.yml` files live inside the directory of the install profile (see the directory structure of core's [Standard](http://cgit.drupalcode.org/drupal/tree/core/profiles/standard?h=8.1.2) and [Minimal](http://cgit.drupalcode.org/drupal/tree/core/profiles/minimal?h=8.1.2) install profiles). Configuration Installer is a way to say "my install profile configuration is actually over in `sites/default/config`." It's a workaround that works well. [Here's a pull request that updates the permissions for the Content Administrator role's configuration and changes some Behat tests accordingly](https://github.com/stevector/nerdologues-d8/pull/67/files).

## How are other people working without Configuration Installer?

I would not need Configuration Installer if I established a canonical Drupal 8 database. With a canonical Drupal 8 database, CircleCI would would grab a copy of the D8 database and import configuration changes rather than installing fresh which creates a new database. Once the Drupal 8 site is live, that will be the workflow. But I don't want to go that workflow yet. A Drupal site has three main parts:

- Version-controlled code.
- Files ignored from version control (mostly uploaded images/documents).
- A database.

The database is easiest element of the three to mess up and the hardest to fix after a mistaken change. That's why Drupal 8 introduced new configuration management tools that move focus from the database to version controlled code. *Configuration does still live in the database* as far as a runtime Drupal site is concerned but configuration is so cleanly imported and exported to code now that mental focus goes to the `.yml` files.

In essence, a Drupal-to-Drupal migration is the transitioning from one set of canonical code/files/database to another. I want to minimize the amount of time that I have two canonical databases to worry about. Right now I can think that any pull request I make on my Drupal 8 code can result in the running of a full data migration to a disposable Drupal 8 database. If there is something I don't like about the Drupal 8 database that is produced by the pull request, I can throw it away and try again. Even if the changes are correct, the Drupal 8 databases are all temporary at this point.

If I did have a canonical Drupal 8 database, it would be running on some tag of my Drupal 8 code's master branch. I would be much more hesitant to make merges to master out of fear that I might mess up the Drupal 8 canonical database. I will put off that conceptual switch as long as doing so saves me time and mental energy. Especially for a side project, I don't want to be thinking with every pull request "in what way exactly is this change made more complicated by having two canonical databases?"


## Where are the relevant pieces in the project?

(These links are to a specific git hash in my Drupal 8 repo because the relevant code might have moved by the time you are reading this blog post.)

* [The Configuration Installer profile lives in the top-level `profiles` directory](https://github.com/stevector/nerdologues-d8/tree/dbb0f71d5d71509d734ad47ab24a6a8f8ec12732/profiles/config_installer). Soon, I'll switch to managing it (and all contrib projects) with Composer and then I won't worry directly about where they go in the folder structure.
* The `.yml` configuration files for the Drupal 8 site live in [`sites/default/config`](https://github.com/stevector/nerdologues-d8/tree/dbb0f71d5d71509d734ad47ab24a6a8f8ec12732/sites/default/config).
* With each pull request, the installation is run [inside CircleCI](https://github.com/stevector/nerdologues-d8/blob/master/circle.yml#L73) (and again in a Pantheon Multidev environment) using `drush site-install config_installer`. 





