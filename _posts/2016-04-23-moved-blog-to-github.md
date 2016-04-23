---
layout: post
title: Moved blog to Github
---

Today I migrated my blog to Github. Although Blogger and LiveJournal were nice, they also left me with a feeling of lack of ownership in my content, and worry about the implications if the service were to be discontinued (especially with Blogger).

In doing a brief search, I found a modern, hacker-friendly technique for [blogging via Jekyll](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/) and hosting with Github pages. Brilliant!

I used the Jekyll Now technique to fork the scaffolding for the blog and immediately had a blog up and running.

Then, I spent a few hours doing the migration. The migration of the blog entries was easy and straightforward. I exported the Blogger blog and then ran the [Jekyll importer](https://import.jekyllrb.com/docs/blogger/) on it, which generated a pile of _posts for my new Jekyll repo. I committed them and like magic, my Blogger entries appeared at the new site (https://jaraco.github.io).

Finally the hard part - migrating comments. Jekyll doesn't handle dynamic content, such as comments, but the beautiful, mashup design of the Internet allows for another service, Disqus, to host and manage the comments (and discussion). Disqus no longer supports importing from Blogger (though it apparently did at one point), though it does support importing from Wordpress, and there is a project to convert from Blogger to Wordpress called "Google blog converters appengine". I haven't linked it because the project is largely defunct and any links are likely to be broken soon. The hosted version of the conversion service is no longer online, and although the code is currently hosted in Subversion on Google archives, I no longer have a good Subversion client.

I was able, however, to download the conversion script from a Github clone of the repository, install the necessary gdata dependency. Still, the script would crash even on Python 2.7. I had to create an environment in Python 2.6 to avoid [the error](https://github.com/vitaliyborodin/google-blog-converters-appengine/issues/77).

After converting the blog to Wordpress format, I was able to import those comments into Disqus, but I found I also had to rewrite all of the `link` elements to remove the leading date in the path (Blogger URLs start with /YYYY/DD). After rewriting the links (using regex replace in SublimeText), I re-imported the comments into by Disqus site and the comments magically appeared with the old blog entries.

More than ever, I own my content and feel imminently empowered to create content, either on the command line or through the web site. I plan to explore using [the Prose editor](http://prose.io/) for easy through-the-web authoring.
