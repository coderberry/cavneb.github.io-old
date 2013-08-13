---
layout: post
title: "GitHub's Copy to Clipboard with Ember"
date: 2013-07-19 10:36
comments: true
categories: 
- EmberJS
---

{% img /images/posts/zeroclipboard-video.gif %}

GitHub recently [replaced the copy and paste functionality](http://techcrunch.com/2013/01/02/github-replaces-copy-and-paste-with-zeroclipboard/) with [ZeroClipboard](http://zeroclipboard.github.io/ZeroClipboard/), a library for copying text to the clipboard that uses an invisible Adobe Flash movie through a JavaScript interface.

I wanted to integrate this functionality into an app that I am writing for [Instructure](http://www.instructure.com) and have been able to do so using very little code.

Here's how I did it originally using Handlebars helpers:

<iframe width="100%" height="500" src="http://jsfiddle.net/cavneb/GSF8Q/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

<p>This is awesome, however I would like to take this a step farther and create a <span style="text-decoration: line-through;">directive</span> Ember web component instead.</p>

<iframe width="100%" height="450" src="http://jsfiddle.net/cavneb/XbDEM/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Ahh.. I feel very good about where [Ember Components](http://emberjs.com/api/classes/Ember.Component.html) are going.
