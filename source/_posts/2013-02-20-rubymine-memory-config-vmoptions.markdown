---
layout: post
title: "What is the best memory config for RubyMine?"
date: 2013-02-20 08:30
comments: true
categories: 
- RubyMine
- IDE
---

RubyMine has become *awesomer* and *awesomer* over the years and is now my primary development tool for Rails development. One thing I've found frustrating however is that there is very little documentation on the best memory settings for your computer.

I'd like to get a thread going here with people identifying their computer, CPU, memory and config settings for RubyMine.

The config file can be located in the `bin` folder of your application. On my mac, the file is `/Applications/RubyMine/bin/idea.vmoptions`. If it doesn't exist you may need to create it.

So here are my computer specs and vmoptions:

MacBook Pro Retina 2.6 GHz Intel Core i7 with 16 GB memory

```
-Xms128m
-Xmx2048m
-XX:MaxPermSize=500m
-XX:ReservedCodeCacheSize=128m
-XX:+UseCodeCacheFlushing
-XX:+UseCompressedOops
```

Please feel free to post yours in the comments below. I believe this will help others optimize their RubyMine experience.
