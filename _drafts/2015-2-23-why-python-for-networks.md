---
title: Why Python for Network Engineers
author: Yandy Ramirez
layout: post
date: 2015-02-23 07:23:17
image: python.jpg
twitter_image: python.jpg
description: 
summary: 
permalink: 
categories:
  - Testing
tags:
  - testing
---
<hr>
![]({{ site.url }}/images/python.jpg)
<br><hr>

Why Python? Simple answer, because if I can do it, then anyone can. But, there's a bit more complicated strategy to this. 

{% highlight python %}
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)
{% endhighlight %}

{% highlight c %}
#include<stdio.h>
 
long factorial(int);
 
int main()
{
  int n = 5;
  long f;
  
  if (n < 0)
    printf("Negative integers are not allowed.\n");
  else
  {
    f = factorial(n);
    printf("%d! = %ld\n", n, f);
  }
 
  return 0;
}
 
long factorial(int n)
{
  if (n == 0)
    return 1;
  else
    return(n * factorial(n-1));
}
{% endhighlight %}