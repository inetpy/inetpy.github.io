---
title: Using Python's Virtual Environments with Git For More Than Coding
author: Yandy Ramirez
layout: post
date: 2015-02-19
image: py_virtualenv.jpg
twitter_image: py_virtualenv.jpg
description: Most think of python virtualenv as a way to create isolated development environments within a single PC, Laptop, Server, whatever their choice is. This is true, it's what virtualenv was created for. 
summary: Most think of python virtualenv as a way to create isolated development environments within a single PC, Laptop, Server, whatever their choice is. This is true, it's what virtualenv was created for...
permalink: 2015/02/python-virtualenv-projects/
categories:
  - Coding
tags:
  - python
  - python2
  - python3
  - python-virtualenv
  - virtualenv
  - virtualenvwrapper
  - virtual-environments
  - projects
---
<hr>
![](http://inetpy.com/images/py_virtualenv.jpg)
<hr>
Most think of **python virtualenv** as a way to create isolated development environments within a single PC, Laptop, Server, whatever their choice is. This is true, it's what **virtualenv** was created for (so they say). 

> This is not a tutorial on virtualenv or virtualenvwrapper, for that visit <a href="http://bit.ly/1zQH84J" target="_blank">this page.</a>

While I do use them for their intended purpose and work on all my **python** projects inside a *virtualenv*.  There's more to them than this as they have further use for all project types in conjunction with **git**.

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- ipy_responsive_2_text -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-2031545302097188"
     data-ad-slot="7544446295"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

### Creating Virtualenvs

There's an aliases file which amongst other things has two lines, one for each version of python.

{% highlight bash %}
$ cat .aliases | grep mkvenv
alias mkvenv3='mkvirtualenv -p /usr/local/bin/python3'
alias mkvenv2='mkvirtualenv -p /usr/local/bin/python2'
{% endhighlight %}

If this is a python project obviously make sure that the correct version of python is chosen for the environment. Otherwise, the version is irrelevant and I always choose the **mkvenv3** alias. Lets say I'm working on a network design for a customer and I have the below files.

{% highlight bash %}
yandy @ YR-rMBP13 in ~/Dropbox/Projects/dummy_design
$ la
total 0
drwxr-xr-x  4 yandy 136 Feb 19 08:51 .
drwxr-xr-x 18 yandy 612 Feb 19 08:50 ..
-rw-r--r--  1 yandy   0 Feb 19 08:50 customer-ip-design.xlsx
-rw-r--r--  1 yandy   0 Feb 19 08:51 customer-net-design.docx
{% endhighlight %}

This is not exactly a python project, but, creating a virtualenv helps me easily switch between projects at the terminal without needing an unreasonable amount of open tabs. I'll explain this in a bit, lets create the virtualenv.

We call the alias **mkvenv3** and give the virtualenv a name.
{% highlight bash %}
$ mkvenv3 dummy-design
Running virtualenv with interpreter /usr/local/bin/python3
Using base prefix '/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4'
New python executable in dummy-design/bin/python3.4
Also creating executable in dummy-design/bin/python
Installing setuptools, pip...done.
Collecting ipython[all]
  Using cached ipython-2.4.1-py3-none-any.whl
Collecting gnureadline (from ipython[all])
.....
[REST OF OUTPUT REMOVED]
{% endhighlight %}

The script that runs after the virtualenv is created has a few **pip install [x]** commands which is why there's a long output. This is not necessary, but I always install a few packages during this process, specifically **ipython[all]**. The **pip install ipython[all]** with the [all] attached makes sure all dependencies are installed.

Once the environment is created and we're in the correct directory, just run the **setvirtualenvproject** keyword.

{% highlight bash %}
$ setvirtualenvproject
Setting project for dummy-design to /Users/yandy/Dropbox/Projects/dummy_design
{% endhighlight %}

This sets the working directory, any time this virtualenv gets activated it will automatically change to the project directory.

### Working with Git

There are plenty of Git tutorials out there, this is one thing I use for most if not all my projects. Can't really beat the simplicity and flexibility of version control.

**Initialize Git**
{% highlight bash %}
$ git init
Initialized empty Git repository in /Users/yandy/Dropbox/Projects/dummy_design/.git/
(dummy-design)yandy @ YR-rMBP13 in ~/Dropbox/Projects/dummy_design
$ git co -b master
Switched to a new branch 'master'
(dummy-design)yandy @ YR-rMBP13 in ~/Dropbox/Projects/dummy_design
$ git add .
(dummy-design)yandy @ YR-rMBP13 in ~/Dropbox/Projects/dummy_design
$ git ca 'initial commit and added files'
[master (root-commit) adc4af5] initial commit and added files
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 customer-ip-design.xlsx
 create mode 100644 customer-net-design.docx
{% endhighlight %}

I branch out from here most of the time or just work on the master, it's really up to you. If you don't know what Git is go <a href="http://bit.ly/1zQJRv0" target="_blank">here.</a>

### Switching Between Projects

There are already a few other environments created, and because of ADHD or whatever excuse I come up with. I don't always work on the same project 100% of the time. 

Now lets say we wanted to work on a specific project, for instance, writing this post on my blog project. It's easy enough, in the terminal just type **workon** followed by a space and the name of the virtualenv. If you can't exactly remember the name, just press space and then tab twice. You'll be given a list of the active virtual environments.

{% highlight bash %}
yandy @ YR-rMBP13 in ~
$ workon [tab twice]
dotfiles      dummy-design      ipyandy-blog
{% endhighlight %}

Choose **ipyandy-blog**
{% highlight bash %}
$ workon ipyandy-blog
(ipyandy-blog)yandy @ YR-rMBP13 in ~/Dropbox/Projects/ipyandy on master*
{% endhighlight %}

The virtualenv gets activated and I'm placed in the correct directory. If I just want to change to another project, just follow the same steps, and...

{% highlight bash %}
(ipyandy-blog)yandy @ YR-rMBP13 in ~/Dropbox/Projects/ipyandy on master*
$ workon dummy-design
(dummy-design)yandy @ YR-rMBP13 in ~/Dropbox/Projects/dummy_design on master
{% endhighlight %}

**SHAZAM**! magic happens.
![](http://inetpy.com/images/shazam.jpg)

### Summary

I found this to be productive enough for me that it merits a blog post. Hopefully you know already, but if you din't now you know and...

> knowing is half the battle - G.I. Joe

Quickly create project environments for programming or other and switch between them at will.