---
ID: 27
post_title: gh test - pr test
author: admin65534
post_date: 2016-01-02 00:32:46
post_excerpt: ""
layout: post

published: false
---

This post is the first in a series that I will write to detail the process I go through to migrate nerdologues.com from Drupal 7 to Drupal 8. I'm doing this work for a few reasons. First and foremost, I'm doing it to stay sharp as a web developer. In my job at Pantheon I draw heavily on my experience building sites in WordPress and Drupal 5, 6, and 7. Drupal 8 has a lot of changes. The more real world experience I have with it, the more effective I'll be.


I've given myself a few parameters for this work:

* No visual changes. I want to be able to run a screenshot-based regression tool like Wraith and show that the D7 and D8 sites are visually identical.
* I'm making this restriction for the following reasons:
  * I have to draw the line somewhere. If I have some visual difference between the sites, it would likely become a slippery slope to complete redesign and I simply do not want to do that.
  *  To show that it is possible. Since I saw it, I have been somewhat stuck on Todd's vision for how CMS will be updated in the future. I want to show that this model, where the backend changes completely and the front-end does not change at all is possible when moving from  D7 to D8. Sure, this site has a very simple visual design, but I'm also just one guy working in spare time.
* I have some ugly Twig files. In Drupal 7 I was using the menu system and other abstractions to generate the header and footer. In Drupal 8 I just want the HTML to match D7 for now. So the header and footer files are just HTML.  
* A Pantheon to Pantheon migration.
* Pull requests for everything.
  * Migrations runable on pull request.
* In a different browser I'm into GitHub as Faux Al Gore, so that I can give myself pedantic code reviews.
* No canonical database yet. All config changes install thanks to config installer profile.
* I'm using github issues to track my work
* Tests run on Circle CI.
* I'm using Behat to test the functionality of a built site inside Circle CI. Behat also runs tests against a Pantheon multidev site built for the pull request to check the results of a migration.
* As much as possible, I'm taking chances to learn new Drupal 8 paradigms including services, plugins for field formatters, ....
* I'm doing local development with `drush rs`.
* I want to refactor my repository so that my pull requests aren't polluted by compiled composer dependencies. But I'll probably wait until Pantheon has more direct support for Drupal Project.

Each of those bullet points could be their own blog post. 