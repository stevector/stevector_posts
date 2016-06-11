---
ID: 35
post_title: 'Drupal 7 to Drupal 8 Migration Diary (Part 2): Using Config Installer to delay a canonical database'
author: admin65534
post_date: 2016-06-05 19:11:06
post_excerpt: ""
layout: post
published: true
---

Drupal sites can be broken down into three component parts.

- Version-controlled code.
- Files ignored from version control.
- Database.

The Drupal 7 site I am migrating from has all three elements and when the Drupal 8 site goes live, it too will have all three elements. Before I started the Drupal 8 project I only had to worry about the three elements on the Drupal 7 site. When I'm done, I will have only three elements to worry about as well (the same three elements in Drupal 8). I know from my experience in my previous job that these



Much of the focus in the development of Drupal 8 was given to moving some concepts from the databased to version-controlled code. The configur Configuration like the definition of content types/fields, the site name 


* I'm using config installer
* What does config installer do
* Why would I want that
* How
* Where are the relevant parts in the code
* Who cares
* When




Before I get into what Configuration installer does, first here is an overview of what happens if you install Drupal from scratch. One of the first things that happens is a prompt for what install profile you would like to use. Drupal Core comes with two install profiles Standard and Minimal. 



## What does Configuration Installer do?

[Config Installer](https://www.drupal.org/project/config_installer) allows a Drupal site to be installed and reinstalled with a pre-existing set of configuration. 




When I started on this project I was excited to use Drupal 8's configuration management system. My imagined workflow was something like

* Install Drupal 8 with the core's Standard install profile.
* Make configuration changes.
* Export configuration changes to code (wooo!) to the `sites/default/config` directory.
* Push committed configuration to GitHub, make pull request.
* CircleCI reinstalls with the Standard profile, import configuration from sites/default/config, run tests.

But there is a problem with this workflow. When attempting to do the import of pre-existing configuration Drupal will complain with a message like :

@todo get error message

As far as Core is concerned, `sites/default/config` is meant for configuration from the same site (or at least different copies of the same site). Doing a fresh install of Drupal core means that the pre-existing config in `sites/default/config` is seen as foreign.

# Enter Configuration Installer

Configuration Installer allows me to use the workflow that I want. Drupal bases new installations on installation profiles. In Drupal 8, each install profile is expected to provide the configuration it needs in the form of `.yml` files. For most every install profile, those `.yml` files live inside the directory of the install profile. See the directory structure of core's [Standard](http://cgit.drupalcode.org/drupal/tree/core/profiles/standard?h=8.1.2) and [Minimal](http://cgit.drupalcode.org/drupal/tree/core/profiles/minimal?h=8.1.2) install profiles. Configuration Installer is a way to say "my configuration is actually over in `sites/default/config`."

# Wait, why do I need this?

I would not need Configuration Installer if I was will to commit to a canonical Drupal 8 database. But I don't want to do that yet. A Drupal site has three main parts:

- Version-controlled code.
- Files ignored from version control.
- A database.

The database is the element that is the easiest part to mess up. That's why Drupal 8 introduced new configuration management tools that move focus from the database to version controlled code.

To do a migration between any versions of Drupal, developers have to transition from a canonical Drupal 7 database to a canonical Drupal 8 database. I want to minimize the amount of time that I have two canonical databases to worry about. Right now I can think that any pull request I make on my code can result in the running of a full data migration to a disposable Drupal 8 database. If there is something I don't like about the Drupal 8 database that is produced by the pull request, I can throw it away and try again. If I did have a canonical Drupal 8 database, it would be running on some tag of my Drupal 8 code's master branch and I would be much more hesitant to make merges to master out of fear that I might mess up the Drupal 8 canonical database.



 as There is a reason the Drupal core community spent so much time in the Drupal 8 development cycle on Configuration Management. It allows for much of our workflow 




 is technically an install profile. I think of it first as the thing I need achieve my desired workflow. The fact that it is an install profile is an implementation detail in my mind. 





