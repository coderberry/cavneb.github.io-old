---
layout: post
title: "GitHub's Copy to Clipboard with Ember"
date: 2013-07-19 10:36
comments: true
categories: 
- EmberJS
---

{% img /images/posts/zeroclipboard-video.gif %}

GitHub recently [replaced the copy and paste functionality](http://techcrunch.com/2013/01/02/github-replaces-copy-and-paste-with-zeroclipboard/) with [ZeroClipboard](http://jonrohan.github.com/ZeroClipboard/), a library for copying text to the clipboard that uses an invisible Adobe Flash movie through a JavaScript interface.

I wanted to integrate this functionality into an app that I am writing for [Instructure](http://www.instructure.com) and have been able to do so using very little code. Here's how I did it.

```javascript app.js
var App = Ember.Application.create();

App.ApplicationController = Ember.ObjectController.extend({
  message: 'This is the text that should be copied'
});

Ember.Handlebars.helper('zeroClipboard', Ember.View.extend({
  tagName: 'button',
  classNames: ['btn'],
  attributeBindings: ['title', 'data-clipboard-text'],
  title: 'Copy to clipboard',

  didInsertElement: function() {
    var clip = new ZeroClipboard(this.$(), {
      moviePath: "/ZeroClipboard.swf"
    });

    clip.on( 'complete', function(client, args) {
      alert('Copied "' + args.text + '" to clipboard');
    });
  }
}));
```

and the HTML:

{% raw %}
```html index.html
<html>
<head>
  <meta charset="UTF-8">
  <link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.no-icons.min.css" rel="stylesheet">
  <script src="http://code.jquery.com/jquery-1.9.1.js"></script>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/handlebars.js/1.0.0-rc.4/handlebars.js"></script>
  <script src="http://builds.emberjs.com.s3.amazonaws.com/ember-1.0.0-rc.6.js"></script>
</head>
<body>

  <script type="text/x-handlebars">
    <div class="container">
      <div class="input-append">
        <span class="span4 uneditable-input">{{message}}</span>
        {{#zeroClipboard data-clipboard-textBinding="message"}}Copy{{/zeroClipboard}}
      </div>
    </div>
  </script>

  <script type="text/javascript" src="ZeroClipboard.js"></script>
  <script type="text/javascript" src="app.js"></script>
</body>
</html>
```
{% endraw %}

I would have used JSBin to share this code, however I couldn't use a SWF file on it, so instead you can download the code:

[zeroclipboard-ember.zip](/assets/posts/zeroclipboard-ember.zip)

