---
post_title: Drupal 7 to Drupal 8 Migration Diary #1: What have I gotten myself into?
author: admin65534
post_excerpt: ""
layout: post

published: false
---

This post is the first in a series that I will write to detail the migration of [nerdologues.com](https://www.nerdologues.com/) from Drupal 7 to Drupal 8. When I lived in Chicago I was an ensemble member with The Nerdologues performing sketch comedy. I've continued maintaining the site since moving to Milwaukee two years ago.

Since building the site withDrupal 7 in 2012 I have used it to test out different areas of Drupal or web development that interest me. [It uses Panels in the way that I like](https://www.palantir.net/blog/explaining-panels-why-i-use-panels) and some of the templates are converted to use Drupal 7 Twig. As soon as Cloudflare unrolled their free plan, I jumped on it to get the site using HTTPS. When I wanted to learn more about Symfony, I wrote a microservice (of sorts) on another server that takes the mp3s of podcast episodes and cuts them up based on timing data entered in Drupal to populate [a clip archive](https://www.nerdologues.com/podcasts/your-stories/clips). 

So it's a typical side project. Some parts of handled with great care. Some are sloppy and out of date. I have put off a Drupal 8 migration until recently because I'm not expecting any huge gains in functionality relative to the amount of work this will take. The members of the Nerdologues who add content might like the improved admin experience of Drupal 8. But it won't be all that different. To limit/clarify scope (and to give me an excuse to do screenshot-based testing) I want the public-facing side of the site to be identical, pixel-for-pixel, to the Drupal 7 site. There are plenty of ways the public-facing side of the site could be improved. And those changes will be easier in Drupal 8.

But will it be worth the time? If I were back in the agency world advising a real client I would say "this Drupal 7 site is fine. Don't upgrade until you have a compelling reason." My personal compelling reason to do the migration anyway is that I want to stay sharp on site building; particularly in Drupal 8. Since switching jobs in September, I'm no longer building sites all the time. Still, my job at Pantheon is much easier when I can speak from direct experience. So to Drupal 8 we go!

In no particular order, here are aspects of the project that may turn into their own blog posts:

* The Drupal 7 site is live on Pantheon. The Drupal 8 version has its own Pantheon site.
* The Drupal 8 code is in [a public GitHub repository](https://github.com/stevector/nerdologues-d8) for anyone who is curious. The Drupal 7 repo will remain private.
* I'm using GitHub issues to track my work.
* All changes to the repo go through a pull request.
* With each pull request, Circle CI does the following:
  * Runs PHP code sniffing and phpunit tests.
  * Installs the Drupal site in the Circle CI container and runs Behat tests against that site.
  * Deploys to a Pantheon multidev environment named for the Circle build number (which is kind of overkill).
  * Runs a migration in the Drupal 8 Pantheon multidev environment, reading from a Drupal 7 Pantheon multidev environment.
* In a different browser I'm signing into GitHub as [Faux Al Gore](https://github.com/fauxalgore), so that I can give myself pedantic code reviews.
* There is no canonical Drupal 8 database yet. All config changes install thanks to [Configuration Installer](https://www.drupal.org/project/config_installer) profile.
* As much as possible, I'm taking chances to learn new Drupal 8 paradigms including services, plugins for field formatters, and the Drupal 8 migration suite.
* I'm doing local development with `drush rs`.
* There are a lot of file fields on the Drupal 7 site that deal with different types of local and remote files. I haven't yet figure out how I will model them in Drupal 8.
* I want to refactor my repository structure and build process so that my pull requests aren't [polluted by compiled Composer dependencies](https://github.com/stevector/nerdologues-d8/pull/103/files). I'll probably wait until Pantheon has more direct support for [Drupal Project](https://github.com/drupal-composer/drupal-project).

If any of these topics sound interesting to you, let me know! Comment on one of the issues in [my repo for managing future blog posts](https://github.com/stevector/stevector_posts/issues) and let me know what I should write about next. 
