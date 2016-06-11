---
post_title: 'Drupal 7 to Drupal 8 Migration Diary (Part 2): Using Config Installer to delay a canonical database'
post_excerpt: ""
layout: post
published: false
---

When I started on this project I was excited to use Drupal 8's configuration management system. My imagined workflow was something like:

* Install Drupal 8 with the core's Standard install profile.
* Make configuration changes.
* Export configuration changes to code (wooo!) to the normal `sites/default/config` directory.
* Push committed configuration to GitHub, make pull request.
* CircleCI reinstalls with the Standard profile, import configuration from `sites/default/config`, run tests.

But there is a problem with this workflow. When attempting to do the import of pre-existing configuration Drupal will complain with a message like :

@todo get error message

As far as Core is concerned, `sites/default/config` is meant for configuration from the same site (or at least different copies of the same site). Doing a fresh install of Drupal core means that the pre-existing config in `sites/default/config` is seen as foreign.

# Enter Configuration Installer

[Configuration Installer](https://www.drupal.org/project/config_installer) allows me to use the workflow that I want. (Thanks to [Alex Pott](https://www.drupal.org/u/alexpott) for writing it!) Drupal bases new installations on installation profiles. In Drupal 8, each install profile is expected to provide the configuration it needs in the form of `.yml` files. For most every install profile, those `.yml` files live inside the directory of the install profile. See the directory structure of core's [Standard](http://cgit.drupalcode.org/drupal/tree/core/profiles/standard?h=8.1.2) and [Minimal](http://cgit.drupalcode.org/drupal/tree/core/profiles/minimal?h=8.1.2) install profiles. Configuration Installer is a way to say "my configuration is actually over in `sites/default/config`." It's a workaround that works. [Here's a pull request that updates the permissions for the Content Administrator configuration and changes some Behat tests accordingly](https://github.com/stevector/nerdologues-d8/pull/67/files). If it  

# Wait, why do I need this?

I would not need Configuration Installer if I established a canonical Drupal 8 database. But I don't want to do that yet. A Drupal site has three main parts:

- Version-controlled code.
- Files ignored from version control (mostly uploaded images/documents).
- A database.

The database is the element that is the easiest part to mess up and the hardest to restore after a mistaken change. That's why Drupal 8 introduced new configuration management tools that move focus from the database to version controlled code. *Configuration does still live in the database* as far as a runtime Drupal site is concerned but it is so cleanly imported and exported to code that mental focus goes to the files.

In essence, a Drupal-to-Drupal migration is the transitioning from one set of canonical code/files/database to another. I want to minimize the amount of time that I have two canonical databases to worry about. Right now I can think that any pull request I make on my Drupal 8 code can result in the running of a full data migration to a disposable Drupal 8 database. If there is something I don't like about the Drupal 8 database that is produced by the pull request, I can throw it away and try again.

If I did have a canonical Drupal 8 database, it would be running on some tag of my Drupal 8 code's master branch. I would be much more hesitant to make merges to master out of fear that I might mess up the Drupal 8 canonical database.


# So where are the relevant pieces in the project?

(These links are to a specific git hash in my Drupal 8 repo because the relevant code might have moved by the time you are reading this blog post.)

* [The Configuration Installer profile lives in the top-level `profiles` directory](https://github.com/stevector/nerdologues-d8/tree/dbb0f71d5d71509d734ad47ab24a6a8f8ec12732/profiles/config_installer). Soon, I'll probably switch to managing it (and all contrib projects) with Composer and then I won't worry directly about where they go.
* The actual `.yml` files live in [`sites/default/config`](https://github.com/stevector/nerdologues-d8/tree/dbb0f71d5d71509d734ad47ab24a6a8f8ec12732/sites/default/config).
* With each pull request, the installation is run [inside CircleCI](https://github.com/stevector/nerdologues-d8/blob/master/circle.yml#L73) (and again in a Pantheon Multidev environment) using `drush site-install config_installer`. 




