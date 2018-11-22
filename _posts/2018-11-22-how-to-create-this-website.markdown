---
layout: post
title:  "How this website was created"
categories: website creation
date:   2018-11-22 16:55:05 +0800
---
This website is build using jekyll and hosted on github pages

I learnt how to 

- install jekyll on windows using this Jekyll [install video][Jekyll-install-video] 
- host jekyll webpage on github pages [host video][github-host-web]. This youtube channel also has many useful videos on creating jekyll webpages
 
 
The following is a summary of the commands used in the video mentioned above

Jekyll commands:
{% highlight jekyll%}
-create new blog
jekyll new ga_blog
cd ga_blog

-serve jekyll website first time on local machine
bundle exec jekyll serve

-serve jekyll website on local machine
jekyll serve

-change _config.yml baseurl to git repo name to host on github;
-remove it to serve on local

{% endhighlight %}

Git commands
{% highlight github %}
git init
git checkout -b gh-pages
git branch
git status
git add .
git commit -m "initial commit"
git remote add origin https://github.com/noaihere/ga_blog.git
git push -u origin gh-pages

{% endhighlight %}


[Jekyll-install-video]: https://www.youtube.com/watch?v=LfP7Y9Ja6Qc
[github-host-web]:   https://www.youtube.com/watch?v=fqFjuX4VZmU&t=39s

