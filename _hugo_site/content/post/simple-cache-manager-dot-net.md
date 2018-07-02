---
aliases:
    - /c#/caching/2015/03/11/simple-cache-manager-dot-net.html
title: "Lightweight Cache Manager for .Net"
date: 2015-03-11
---

Simple cache manger came out of a need to get rid of some bulky libraries to do the simple caching of data. It's only a couple of classes and easy to just copy-paste into your project.

<!--more-->

Here is the API:

{% highlight c# %}
var cacheManager = new CacheManager<string>();
cacheManager.Add("key1", "value1", DateTime.UtcNow, TimeSpan.FromSeconds(300));
cacheManager.Add("key2", "value2", DateTime.UtcNow, TimeSpan.FromSeconds(600));

Assert.True(cacheManager.Contains("key1"));
Assert.True(cacheManager.Contains("key1"));

Assert.Equal("value1", cacheManager.GetData("key1"));
Assert.Equal("value1", cacheManager.GetData("key1"));
{% endhighlight %} 

It's as simple as that. Note the fact you need to pass the DateTime Now as a parameter? This is to make the testing easy. Maybe an overload can be used there.

Internally it is a thin wrapper around a dictionary. Full code is in my [mtmk GitHub repository with the same name as the post](https://github.com/mtmk/simple-cache-manager).

Enjoy