---
layout: post
title: "Using Flash Messages with EmberJS"
date: 2013-06-20 15:12
comments: true
categories: 
- EmberJS
---

<div style="width: 250px;
     height: 338px;
     float: left;
     margin-right: 20px;
     background: transparent url(/images/flash-to-flash.png) 0 0 no-repeat;">
</div>

In a recent app I have written in Ember, I found that the need to display flash messages came up. This has always been given to me for free with Rails, so I didn't really think of it not being just as easy to integrate using Ember. I was wrong.. well, sort of wrong.

On searching the web for 'ember flash', I found a nice little library at [https://github.com/cheapRoc/ember-flash](https://github.com/cheapRoc/ember-flash) by **cheapRoc**. After some minor tweaking and customization, I was able to get it working in my app.

In this post, I want to share how easy it is to integrate and offer a little explanation as to how it works.

Instead of creating four different files (controller, message, queue and view), I found that placing all of the same file made sense. I created the file *flash.js*:

```javascript flash.js
App.FlashMessage = Ember.Object.extend({
  type: "notice",
  message: null,
  isNotice: (function() {
    return this.get("type") === "notice";
  }).property("type").cacheable(),
  isWarning: (function() {
    return this.get("type") === "warning";
  }).property("type").cacheable(),
  isError: (function() {
    return this.get("type") === "error";
  }).property("type").cacheable()
});

App.FlashView = Ember.View.extend({
  contentBinding: "App.FlashController.content",
  classNameBindings: ["isNotice", "isWarning", "isError"],
  isNoticeBinding: "content.isNotice",
  isWarningBinding: "content.isWarning",
  isErrorBinding: "content.isError",
  
  didInsertElement: function() {
    this.$("#message").hide();
    return App.FlashController.set("view", this);
  },
  
  show: function(callback) {
    return this.$("#message").css({
      top: "-40px"
    }).animate({
      top: "+=40",
      opacity: "toggle"
    }, 500, callback);
  },
  
  hide: function(callback) {
    return this.$("#message").css({
      top: "0px"
    }).animate({
      top: "-39px",
      opacity: "toggle"
    }, 500, callback);
  }
});

App.FlashController = Ember.Object.create({
  content: null,
  clearContent: function(content, view) {
    return view.hide(function() {
      return App.FlashQueue.removeObject(content);
    });
  }
});

App.FlashController.addObserver('content', function() {
  if (this.get("content")) {
    if (this.get("view")) {
      this.get("view").show();
      return setTimeout(this.clearContent, 4000, this.get("content"), this.get("view"));
    }
  } else {
    return App.FlashQueue.contentChanged();
  }
});

App.FlashQueue = Ember.ArrayProxy.create({
  content: [],
  contentChanged: function() {
    var current;
    current = App.FlashController.get("content");
    if (current !== this.objectAt(0)) {
      return App.FlashController.set("content", this.objectAt(0));
    }
  },
  pushFlash: function(type, message) {
    return this.pushObject(App.FlashMessage.create({
      message: message,
      type: type
    }));
  }
});

App.FlashQueue.addObserver('length', function() {
  return this.contentChanged();
});
```

## How does it work?

A **FlashMessage** is an Ember object *(lines 1-13)* which contains the message text and type of message (notice, warning, error). These messages are added into the **FlashQueue**, which is an array proxy, via the *pushFlash* function *(lines 75-59)*. There is an observer *(lines 83-85)* which triggers the function *contentChanged* whenever a message is added to the queue. When the *contentChanged* function is called, it places the flash message into the **FlashController**'s content *(lines 68-74)*. There is an observer watching the content of the controller *(lines 55-64)* which displays the **FlashView** for a short period of time, then hides it *(lines 58-59)*. Once hidden, *clearContent* is called which hides the view and removes the message from the queue *(lines 46-53)*.

## CSS and Animation

On lines 27-43 above, there are animations set up for when the view is to be shown and hidden. These can be modified to anything. What happens in this code is the view will slide down and fade into view. The reverse happens when it is hidden.

Here is the CSS that I am using:

```css
#flash-view {
  position: fixed;
  top: 100px;
  width: 100%;
  z-index: 20;
  overflow: hidden;
}
#flash-view #message {
  color: #fff;
  font-size: 16px;
  line-height: 1.2em;
  text-align: center;
  padding: 20px 0;
}
#flash-view.is-notice #message {
  background-color: #6c3;
}
#flash-view.is-alert #message {
  background-color: #f14247;
}
#flash-view.is-error #message {
  background-color: #f14247;
}
```

## Usage

First, you must include the JavaScript and CSS above in your app.

Second, add the view into your template (e.g. application.handlebars)

{% raw %}
``` javascript application.handlebars
{{#view App.FlashView id="flash-view"}}
  <div id="message">
    {{view.content.message}}
  </div>
{{/view}}

<h1>Application</h1>
{{outlet}}
```
{% endraw %}

Finally, trigger messages to be shown with the *pushFlash* function:

    App.FlashQueue.pushFlash('notice', 'This actually works!!!');

<a class="jsbin-embed" href="http://jsbin.com/olurim/5/embed?live">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>