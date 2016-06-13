---
ID: 53
post_title: 'Drupal 7 to 8 Migration Diary (Part 3): Regression found'
author: admin65534
post_date: 2016-06-12 21:28:29
post_excerpt: ""
layout: post
permalink: >
  http://stevector.com/2016/06/drupal-7-to-8-migration-diary-part-3-regression-found/
published: true
---
*This post is the third in a series about migrating [nerdologues.com](https://www.nerdologues.com/) from Drupal 7 to Drupal 8. [It is a direct follow-up to yesterday's post about migrations and canonical databases](http://stevector.com/2016/06/drupal-7-to-8-migration-diary-part-2-using-configuration-installer-to-delay-a-canonical-database/).*

## TLDR;
* [Yesterday I wrote about how pleased I was with not yet having a canonical Drupal 8 database and migration tests](http://stevector.com/2016/06/drupal-7-to-8-migration-diary-part-2-using-configuration-installer-to-delay-a-canonical-database/).
* One of my migrations (taxonomy terms) stopped working between 8.1.1 and 8.1.2.
* I caught the failure with [a Behat test running in CircleCI](https://circleci.com/gh/stevector/nerdologues-d8/245).
* I restructured my repo using [Drupal Project](https://github.com/drupal-composer/drupal-project) which made it easier to checkout out different commits between 8.1.1 and 8.1.2 to find [the exact problem commit](https://www.drupal.org/node/2692373#comment-11192739).

## The full story
Right after I published a blog post yesterday about [putting off a canonical Drupal 8 database](http://stevector.com/2016/06/drupal-7-to-8-migration-diary-part-2-using-configuration-installer-to-delay-a-canonical-database/) I hit a bug that challenged that decision. Upgrading the Drupal 8 codebase to the latest stable version (8.1.2) resulted in [an unexpected Behat failure related to taxonomy term migration](https://circleci.com/gh/stevector/nerdologues-d8/245). I ran the migration locally, signed into to the site, and indeed there were no taxonomy terms listed in the UI. But `drush migrate-status` told me that 74 terms had been migrate. And the taxonomy database tables had records. Odd.

Wanting to get on with my Saturday, I decided to make a new directory for failing tests so that CircleCI could pass with the still-working tests. I reopened [my original GitHub issue which tracked taxonomy migration](https://github.com/stevector/nerdologues-d8/issues/9#issuecomment-225392100). If I had a canonical Drupal 8 database, these taxonomy terms would already be in that database and I doubt I would have noticed this bug.

### Investigation

Coming back to the issue later night I tried a few things:

* I looked through the Drupal core and migrate-related contrib issue queues. [I even found the issue opened in response to the bug I eventually found](https://www.drupal.org/node/2744639). The patch didn't work for me. Not realizing how close I was to the problem, I kept digging.
* I tried just updating the migrate-related contrib modules.
* I went back to 8.1.1. That worked!
* I did a `git diff` on the tags. I looked at the changes in migrate module. Nothing jumped out.

### Isolating the problem commit

I decided to try just checking out commits between the 8.1.1 and 8.1.2. Easier said than done. My repo does not have a direct shared history with Drupal.org since it is from [Pantheon's Drops 8 repo](https://github.com/pantheon-systems/drops-8/commits/master). It has commits per core tag rather than bringing over every single commit from Drupal.org. [Pantheon's Drupal 7 repo](https://github.com/pantheon-systems/drops-7/commits/master) *does* bring over each core commit which I always found more distracting than helpful.

### Composer-based "Drupal Project" to the rescue

Needing a way to run my Drupal 8 site on more granular commits between 8.1.1 and 8.1.2 I decided use the situation as an excuse to restructure my repo with the [Composer-based Drupal Project](https://github.com/drupal-composer/drupal-project). (Sure, I could have just switched over to the Drupal.org repo, but what's the fun in that?) Drupal Project treats Drupal core as a dependency of a project's repo rather than as being the whole project repo. With Drupal Project I knew I could use Composer to specify the exact core commits I wanted to run. [So I made a new branch with the Drupal Project structure](https://github.com/stevector/nerdologues-d8/pull/108) and I replaced the `"drupal/core": "~8.0"` with `"drupal/core": "8.1.x-dev#a5d3f14db87ef953d227fb411489bada1afbb0a8"`. [That hash is from a the subtree split of Drupal core](https://github.com/drupal-composer/drupal-core) that Drupal Project uses rather than from Drupal.org's own repo. I could update that hash  and run `composer update` to jump to specific points in history and then rerun my install and tests locally with a series of terminal commands I built up as I went:

```
chmod 777 web/sites/default/settings.php
cd web
drush si -y config_installer
drush cr
drush mi --all
drush uli --uri=http://127.0.0.1:8888/
cd ..
./vendor/bin/behat --config=private/testing/behat/behat-local.yml   private/testing/behat/features/failing-migration-tests/ --strict
```

This task was made more difficult by migrations failing completely due to a bug in [a different migrate module change that was committed and reverted between 8.1.1 and 8.1.2](https://www.drupal.org/node/2694391). But that's the reality of using experimental core modules. Eventually I realized I didn't even need to keep run `composer update` to move between hashes. (I think) Because Composer saw core as requiring a development branch, it brought the full history of the subtree split into my local copy so that I could `cd` into the `web/core` directory and `git checkout` specific commits.

### Bugs come with the territory

Maybe I should have written this into yesterday's blog post, but running into these kinds of bugs is something I expected. [Migrate is still an experimental module](https://www.drupal.org/core/experimental). It is no surprise that an experimental module has bugs. Running my personal projects in such a way that they find bugs missed by core's tests is a way for me to find core issues where I can help.

## What now?

I'm not sure. I don't want to merge the restructuring of the repo as a Drupal Project into master until Pantheon has more direct support for Drupal Project. [Lucky for me, that's a topic I get to work on as part of my job there](https://github.com/pantheon-systems/documentation/issues/1620).

It is not critical for me to have this taxonomy bug fixed right now. Maybe I'll post a patch that simply reverts the core commit that added the bug. I don't necessarily expect that revert to be committed, but I, and anyone else blocked by it, could use the patch. [I've commented on the issue that should fix this bug](https://www.drupal.org/node/2744639#comment-11286811). But I don't even understand yet how the code change in question resulted in the taxonomy migration failing.

I think the main lesson I'm learning here is that I do still need regression tests for my migrations. At MidCamp a few years ago I did a presentation on using a hacked SimpleTest to regression test migrations to Drupal 7. Writing migration tests in SimpleTeest was inefficient and cumbersome. But Behat feels like the wrong tool too. [The README file for my directory of Behat tests explains some of my thoughts on potential Behat misuse](https://github.com/stevector/nerdologues-d8/blob/2c46df8e61e873bf18144ee5720c02f45a657126/private/testing/behat/features/README.md). I'll expand that into a blog post at some point. Until then, Behat has shown it can catch migration regressions and I'll keep rolling with that.