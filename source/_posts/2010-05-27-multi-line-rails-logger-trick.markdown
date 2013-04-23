---
layout: post
title: "Multi-Line Rails Logger Trick"
date: 2010-05-27
comments: true
categories: 
- Ruby
---

One thing that I like to do with my logger output is allow for multi-line output. It makes it much easier to read and keeps my code cleaner.

Instead of doing this:

``` ruby
logger.info "Current Contest: #{@contest.name}"
logger.info "Current User: #{@contest.name}"
```
    
You can do this:

``` ruby
logger.info """
    Current Contest: #{@contest.name}
    Current User: #{@contest.name}
"""
```