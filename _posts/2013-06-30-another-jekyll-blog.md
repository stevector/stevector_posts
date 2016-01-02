---
ID: 11
post_title: Another Jekyll blog
author: admin65534
post_date: 2013-06-30 18:46:22
post_excerpt: ""
layout: post
permalink: >
  http://live-stevector.pantheon.io/2013/06/another-jekyll-blog/
published: true
---
I'm jumping on the bandwagon. My vague ambitions to start a blog have combined with my desire to build something outside of <a href="https://drupal.org">Drupal</a> (the technology that has been at the center of my job for six years). So like <a href="http://portal.ehawaii.gov/page/developers/">The state of Hawaii</a>, <a href="http://developmentseed.org/blog/new-healthcare-gov-is-open-and-cms-free/">the federal government</a> and Drupal developers <a href="http://angrylittletree.com/">Jeff Eaton</a>, and <a href="http://engineeredweb.com/blog/why-switched-to-jekyll/">Matt Farina</a> I've turned to Jekyll.

From <a href="http://jekyllrb.com/docs/home/">the Jekyll docs page</a>:
<blockquote>Jekyll is a simple, blog-aware, static site generator. It takes a template directory containing raw text files in various formats, runs it through Markdown (or Textile) and Liquid converters, and spits out a complete, ready-to-publish static website suitable for serving with your favorite web server.</blockquote>
That's a lot of jargon that boils to two principles that appeal to me.
<h3>1. Only think about something when you have to think about.</h3>
For a webpage like this one to get to a browser it is necessary for a computer to stitch together these words along with some template code and stylesheets. Jekyll does that work as I write and save a post. Drupal and most other modern site-building tools will do that work over and over again as more people ask for the webpage. The "over and over again" approach allows a site to be slightly different for each logged-in user or to constantly update something like the "most popular stories" list. Losing the ability to "think" every time a page is requested feels like I've thrown out the majority of my toolbox as a web developer. And that's ok. For a site this simple it's not necessary to carry around those heavy tools to constantly rebuild a site that's not actually changing very often.
<h3>2. Version controlling content</h3>
Jekyll keeps the contents of blog posts in text files (Markdown in most cases, mine included). That means the same tool (Almost certainly <a href="http://git-scm.com/">git</a>) used for managing changes to the code of the web site can be used for tracking changes to the content. And the tools for managing code have become wonderful in the past few years. <a href="http://www.ted.com/talks/clay_shirky_how_the_internet_will_one_day_transform_government.html">Clay Shirky's TED talk on git</a> is the best explanation I've seen of why git is worth noticing. For one thing, controlling content through git means that a person can <a href="https://github.com/eaton/eaton.github.com/commit/fd2b2b5ab42296cb803c6fca19e944ab60a061bc">fix a typo in someone else's blog</a> as easily as they'd correct a bug in open source code. Managing content with git will produce other unexpected modes of collaborating and I'm exciting to have a blog that can be a part of this shift.