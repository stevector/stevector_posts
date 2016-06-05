---
ID: 20
post_title: 'Make your Drupal 8 theme easier to maintain with this one weird trick (Twig&#8217;s &#8220;extends&#8221; concept)'
author: admin65534
post_date: 2013-10-27 19:22:08
post_excerpt: ""
layout: post
permalink: >
  http://stevector.com/2013/10/make-your-drupal-8-theme-easier-to-maintain-with-this-one-weird-trick-twigs-extends-concept/
published: true
---
<h3>WHAT is Twig's "extends" concept?</h3>
Drupal 8 has a new templating system called <a href="http://twig.sensiolabs.org/">Twig</a> that comes from the <a href="http://symfony.com/">Symfony</a> world.
I recommend learning about Twig and Drupal 8's use of it by watching a video of one of the many recent conference presentations on the topic like <a href="https://portland2013.drupal.org/session/using-twig-new-template-engine-drupal-8">this one from Drupalcon Portland</a>.
Twig contains within it a different model for overriding templates that can be used in combination with Drupal's traditional naming-based template overrides.
<a href="http://twig.sensiolabs.org/doc/tags/extends.html">Read Twig's own documentation of "extends" here</a> to see how it allows child templates to be much simpler than their parents. Or just keep reading until the 'HOW' section of this post.
<h3>WHY is it helpful?</h3>
Last weekend I was working on a personal Drupal project (<a href="http://nerdologues.com">nerdologues.com</a>) and ran into a common theme development pain point.
I wanted to print some fields separate from the rest of the <code>$content</code> variable for only one node type.
In Drupal 7 (and prior versions) this process involves making a complete copy of the <code>node.tpl.php</code> file (<a href="https://drupal.org/node/17565">See the drupal.org documentation of this process here</a>).

Copying every line of <code>node.tpl.php</code> to <code>node--article.tpl.php</code> only to change a few lines makes the theme of a given Drupal site extremely WET (<a href="http://en.wikipedia.org/wiki/Don't_repeat_yourself#DRY_vs_WET_solutions">Write Everything Twice</a>).
What if instead of duplicating the entire node template file to a node-type specific name, I could make that node-type specific file contain only the overrides?

That's what Twig "extends" concept does!
<h3>HOW is it used?</h3>
Here's an example <code>node--article.html.twig</code> that moves field_image.
<pre>{% extends "themes/sub_bartik/templates/node.html.twig" %}

{# Override the header_fields block to put field_image there because this site
   needs it there. #}
{% block header_fields %}
  {{ content.field_image }}
{% endblock %}

{# Override the content block to hide the field_image subvariable because
   it is being rendered in header_fields. #}
{% block content %}
  {% hide(content.field_image) %}
  {{ content }}
{% endblock %}

</pre>
<a href="https://github.com/stevector/stevector.github.io/blob/example--twig-extends/templates/node.html.twig">Compare that to the amount of code in the parent template being overridden.</a>

To make this kind of overriding possible, the parent template needs to declare which sections can be overridden. In this example I've declared two "blocks" (the Drupal community might start calling these pieces "codeblocks" to reduce confusion with Drupal core's Block module) that designates which pieces of the template can be replaced.

Here's one addition to <code>node.html.twig</code> in a copy of Bartik's version:
<pre>{# This empty block allows child templates to insert markup into this
   place in the header without re-writing the entire template. #}
{% block header_fields %}
{% endblock %}

</pre>
Addition two in <code>node.html.twig</code> is:
<pre>{# By wrapping the content variables in a block, this template allows child
   templates to insert markup into this spot without re-writing the entire
   template. #}
{% block content %}
  {{ content }}
{% endblock %}
</pre>
<h3>WHERE does this code go?</h3>
The complete example code, which is a sub-theme of Bartik, is here <a href="https://github.com/stevector/stevector.github.io/tree/example--twig-extends">in an alternate branch of the github repo of this blog you're reading</a>.
To test the theme <code>git clone</code> that branch into <code>/themes</code> of a Drupal 8 site and enable it.
Notice that the "parent" <code>node.html.twig</code> is in this sub-theme.
By having a <code>node.html.twig</code> in the sub-theme, Drupal (and Twig) completely ignore the <code>node.html.twig</code> in Bartik and node module.
That kind of overriding, where a template file in the theme completely supersedes Core, is the same as previous versions of Drupal.
<h3>WHO should write these "blocks"?</h3>
This example illustrates how "extends" can be used by a site owner in a <strong>site-specific theme</strong>.
I have shown this code around at <a href="http://2013.badcamp.net/">BADCamp</a> this weekend and the reaction from <a href="https://twitter.com/c4rl">Carl Wiedemann</a> and other Twig-interested-developers has mainly been something like "Drupal Core shouldn't put these blocks in templates yet (or maybe ever) because we don't have enough experience with them."
I agree.
Drupal has a pattern of proving concepts in the contrib space and then moving those concepts into core.
And even before making a contrib base theme using Twig blocks, someone has to use them on a real site.
<h3>WHEN will developers start using "blocks"?</h3>
Heavy use of the "extends" concept in Twig will start when developers start building Drupal 8 sites and adding site-specific themes.
<a href="https://twitter.com/Cottser">Scott Reeves</a> pointed out to me that <a href="https://twitter.com/jenlampton">Jen Lampton's</a> experimental Twig base theme for Drupal 8 <a href="https://github.com/jenlampton/twiggy/blob/master/templates/node.html.twig#L98">used this concept already</a>. I'm glad I'm not the only Drupal developer who wants to use this functionality.
<h3>WHAT is next?</h3>
Drupal developers will bikeshed extensively on a rubric for when a block (or is it a codeblock?) should be completely empty in the parent template as I did with my <code>header_fields</code> block and Jen did with her <code>infobar</code> block.
When should the parent template's block contain something as I did with my <code>content</code> block?
Should these blocks be named with an underscore like <code>header_fields</code> or without like <code>infobar</code>?
Is it too confusing for Drupal to add another concept for overriding a template? I have no "right" answers yet.
I want to use this tool for real first and see what new pain points we get while solving the existing ones.