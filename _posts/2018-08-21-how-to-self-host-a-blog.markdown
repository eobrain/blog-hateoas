---
layout: post
title:  "How to self-host a blog"
date:   2018-08-21
categories: Programming
---

One of the purposes of this blog is to experiment with simple ways put up a blog
on the web without depending on any blogging platform. For this I wanted a free
way to deploy static web pages on my own domain name. For this I chose [Firebase Hosting][1],
partly because I am very familiar with it, having worked on the Firebase team
and personally knowing many of the Hosting development team, but also because it
is actually the best solution I know of for what I want, and it is free for
sites that have the traffic of typical personal sites.

People who are comfortable using the command line might be interested in
following along the steps I took.
  
1.  Go to http://firebase.com, login, and create a project.</li>
1.  Install Node. If you don’t already have it installed already, I suggest
    first installing [nvm][2] then doing:

	  ```sh
    nvm install stable
    nvm use stable</pre>
    ```
1.  Follow the instructions in the [Hosting quickstart][3].
1.  Replace `index.html` with the HTML you want as your home page.
1.  Do `firebase deploy`
1.  I’d recommend storing everything in Git, and you might as well make it a
    public repo, as the web site is public anyway. (Mine is [on GitHub][4])
1.  You also might want to register a domain name, and use Firebase Hosting to
    use that domain name (complete with a free SSL certificate so you the
    `https:` security for free.). I did this for `eamonn.org`

[1]: https://firebase.google.com/docs/hosting/
[2]: https://github.com/creationix/nvm
[3]: https://firebase.google.com/docs/hosting/quickstart
[4]: https://github.com/eobrain/webhome
