---
layout: post
title: Using Github pages but hosting elsewhere
keywords: github, pages, jekyll
description: How I used the Github hosted pages service to render my blog.
---

[Github Pages](http://pages.github.com/) are fantastic.

Seriously, I just push into my repository and their service renders the templates
and blog and they host it for free. What could be better than that?

Using your own domain name. On Pages this is restricted to paid account holders.

Well, I'm not a paid account holder but I do have a domain and hosting elsewhere
that I'm already paying for so I figure I can probably use that.

Github provides callback hooks which make a post request of your choice after
pushing. I pointed this callback hook at a simple PHP script on my domain
which uses wget to mirror the site.

Okay, it's not the prettiest but it works and it's all automated. I do nothing
beyond pushing my repository to Github.

Here's the script:
{% highlight php %}
<?php

echo("Updating local cache");
ob_end_clean();

ob_start();

sleep(60);
system('wget -m http://morgangrubb.github.com', $retval);
var_dump($_SERVER);

mail("awesome@sofarsogood.co.nz", "Blog updated - {$retval}", ob_get_clean());

ob_end_clean();

?>
{% endhighlight %}


