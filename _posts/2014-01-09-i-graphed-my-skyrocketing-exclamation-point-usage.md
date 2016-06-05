---
ID: 23
post_title: "I graphed my skyrocketing exclamation point usage!<script>alert('XSS')</script>"
author: admin65534
post_date: 2014-01-09 19:24:50
post_excerpt: ""
layout: post
permalink: >
  http://dev-stevector.pantheonsite.io/2014/01/i-graphed-my-skyrocketing-exclamation-point-usage/
published: true
---
<h3>WHAT is the deal?</h3>
I parsed over three years of my Google Voice data to look for exclamation points in my sent messages. This graph shows what I found.

<iframe src="/assets/post_specific/2014/01/google_voice_exclamation/chart.htm" width="700px" height="410px" frameborder="0"></iframe>
<h3>WHY did I do that?</h3>
My girlfriend and I had talked about how nearly all of our messages contain exclamation points.
I had chatted with friends about exclamation points’ omnipresence.
And then <a href="https://twitter.com/bencrair">Ben Crair</a> wrote <a href="http://www.newrepublic.com/article/115726/period-our-simplest-punctuation-mark-has-become-sign-anger">a story in The New Republic</a> that brought some more high profile ancedata about exclamation points as “sincerity markers.”

After all that conversation I figured that this question about how much exclamation usage was increasing could be answered concretely.
So I downloaded my full Google Voice history, parsed it and graphed what I found.
<h3>WHO was I texting?</h3>
Mostly my girlfriend. We've been together since late October of 2012.
I suspect her frequent exclamation usage has influenced me.
Maybe I'll refactor my code a little to separate out the percentage of text messages sent to her that get exclamation points as compared to everybody else.
I should also note that not every text conversation I have runs through Google Voice.
While the vast majority do, some friends and family still text me directly on my cell number.
<h3>WHAT does it matter?</h3>
The English language has a de facto standard of being controlled by its users.
Unlike French, with its <a href="http://en.wikipedia.org/wiki/Acad%C3%A9mie_fran%C3%A7aise">Académie française</a>, English is driven by ordinary people.
<a href="http://www.economist.com/blogs/johnson/2010/06/english_academy">That's not going to change</a>.
In English, dictionary and style guide publishers chase free publicity by announcing newly added words and <a href="http://artsbeat.blogs.nytimes.com/2013/11/19/selfie-trumps-twerk-as-oxford-dictionaries-word-of-the-year/?_r=0">words of the year</a>.
The ease with which linguistic trends can now be identified will only speed up this process.
<a href="http://www.prdaily.com/Main/Articles/15033.aspx#">In 2013 literally stopped meaning literally</a> and soon will we literally be able to pinpoint when a meaning shifts.
For the editor who wants to rewrite the textbook meaning of exclamation points, the evidence is mounting.
<h3>HOW did I do that?</h3>
First, I <a href="http://techcrunch.com/2011/09/06/google-now-lets-you-export-google-voice-data/">exported my Google Voice history with Google Takeout</a>.
Then I wrote a couple of PHP classes to parse the files in arrays that could be manipulated more easily.
Those classes then went into a <a href="http://symfony.com/">Symfony</a> bundle that used <a href="https://github.com/Malwarebytes/Altamira">Altamira</a> to display the data through <a href="http://www.jqplot.com/">jqPlot</a>.
I've got <a href="https://github.com/stevector/stevector.github.io/pull/9">an open pull request against my blog</a> with an in-progress post that has more details on how the code works.
This code was the first time I wrote a Symfony bundle or app so there's plenty of refactoring to be done.
Pull requests and pedantry are welcome.
<h3>WHERE can the work be checked?</h3>
Well, I don't like the NSA reading my texts and I don’t want you reading them either.
So to a certain extent you’ll just have to trust me.
Jump into a conversation with me in the comments of this post or in the pull request for the upcoming blog post if you have ideas for how such data could be shared in a sanitized fashion.