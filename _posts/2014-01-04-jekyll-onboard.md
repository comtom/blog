---
layout: post
title: Jekyll on board
---

*New year, new blogging plattform*

-----

I've been using Wordpress for a while, but it was a hedeache to update and I didn't use a lot of features, I always felt that it was too big for me. Then I found Tumblr, wich worked fine, but also it was a little big, since I didn't use the social features. Also Tumblr had a lot of stability issues.

As you probably know, less is more, so I choose [Jekyll](http://jekyllrb.com) this time, the static site generator. It's a small but it makes a site very fast and reliable as it generates all html only when you decide it. It includes among other features templates, pages and posts, nothing really to envy to Wordpress or others <bigger> competitors as you don't need to use a database or run a script on each visit.

### Drawbacks

Jekyll does not provide a commments system, so I chose Disqus, a free comments service that it's realy easy to integrate, and it's really nice to use.

### More software

I've found Poole, a more easy way to implement Jekyll. It's developed on and hosted with GitHub. Head to the <a href="https://github.com/poole/poole">GitHub repository</a> for downloads, bug reports, and features requests.

### Why?

Because I'm used to work with git repositories, and all modifications are tracked, so any mistake would be easily recovered, also I can use the repository as a backup, so the same process of modifing content will record that change on the git database and each time I make a push, that data will be stored in a remote host. When the push is finished, a git hook is triggered and then all the files are updated in the webserver so with a very simple to use system, I can resolv the three problems: backup generation, deploying website updates or new content and keeping track of what is modified (and who did that modification).

Hope you enjoyed this article, keep reading.
