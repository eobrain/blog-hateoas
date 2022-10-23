---
layout: post
title:  "Simplest possible web templating?"
date:   2018-10-06
categories: Programming
---

*[See [old version of this post][5] to see what is being described.]*

So, I started off trying to have this blog using simple plain hand-written HTML.
However, I soon start getting annoyed how much repeated boilerplate I had to
write, such as in the `&lt;head` and in linking together the articles from the
main index page. So in an effort to make things more DRY I considered what
simple templating systems I could used.

A long time ago, back before the web existed I used the [M4 macro language][1].
It has the advantage of being installed on my home Linux machine by default, so
no extra dependencies are required. Let's see if we can use it as an HTML
templating language.

You can see an example of a [template using M4][2] in the source code of this
blog. Note that there is just one simple line of boilerplate at the top and some
closing parenthesis at the bottom, but otherwise the template is straightforward
HTML.

```m4
    ARTI{}CLE( 2, &#x7B;

    ...HTML goes here...

    &#x7D;)
```

The templates uses two M4 files:

1.  [lib.html][3] which contains the HTML header and footers for all the
    pages.
  
2.  [data.m4][4] which contains the metadata for the pages, including the
    information needed to create the home page and to provide "previous" and
    "next" links on the article pages.

[1]: https://www.gnu.org/software/m4/manual/m4.html
[2]: https://github.com/eobrain/webhome/blob/75e9de1daa37c5129c10a6cd05074326d41a27ec/templates/2018-08-21.html
[3]: https://github.com/eobrain/webhome/blob/75e9de1daa37c5129c10a6cd05074326d41a27ec/templates/lib.html
[4]: https://github.com/eobrain/webhome/blob/75e9de1daa37c5129c10a6cd05074326d41a27ec/templates/data.m4
[5]: /old/2018-10-06.html
