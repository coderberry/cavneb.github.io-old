---
layout: post
title: "Airbrake exception logging in your Grails application"
date: 2012-09-11 14:34
comments: true
categories: 
- Grails
---

Airbrake has a Java library that can be used in your Grails application fairly easily. A couple of items that the library doesn't do is make it simple to pass session / request data to Airbrake. I also found that the backtrace is somewhat jumbled when viewing it on Airbrake. [Phuong LeCong](https://github.com/plecong/grails-airbrake) wrote an excellent plugin to enhance the library. I have recently published an [updated version](https://github.com/cavneb/airbrake-grails) of the plugin (with Phuong's consent) to the [Grails plugin repository](http://grails.org/plugin/airbrake).

Let's take a look at how simple it is to use.

### Installation

Installation is very simple. Simply add the plugin to your `BuildConfig.groovy` file: 

```
plugins {
    runtime ":airbrake:0.4"
}
```

Next time your app compiles, the plugin should be installed and ready to go.

### Integration

It's nearly as simple to integrate the notification process into your Grails application. This is done by adding a log4j appender into your `Config.groovy` file. Make sure that you update the appender config with the correct project API key from Airbrake.

```
def isProd = Environment.current == Environment.PRODUCTION

log4j = {
  // Example of changing the log pattern for the default console appender:
  appenders {
    appender grails.plugins.airbrake.AirbrakeAppender (
      name: 'airbrake', 
      api_key: 'API_KEY', // replace with your Airbrake API key
      env: (isProd ? 'production' : 'development'),
      enabled: true
    )
    ...
```

After the appender is in place, you need to let your Grails app know to use it as a global appender. Add `'airbrake'` to your root `debug` node:

```
def isProd = Environment.current == Environment.PRODUCTION

log4j = {
  // Example of changing the log pattern for the default console appender:
  appenders {
    appender grails.plugins.airbrake.AirbrakeAppender (
      name: 'airbrake', 
      api_key: 'API_KEY', // replace with your Airbrake API key
      env: (isProd ? 'production' : 'development'),
      enabled: true
    )
    ...
  }

  root {
    debug 'stdout', 'airbrake' // This can be added to any log level, not only 'debug'
  }
}
```

Once this is all in place, you should be able to test it out.

### Testing

To test the Airbrake exception notification, an exception must be thrown in your application when running. The plugin offers a simple way to generate an exception without mucking around with your code.

To force an exception, run your application and visit [http://localhost:8080/airbrakeTest/throwException](http://localhost:8080/airbrakeTest/simulateError). You should see a page similar to this:

<div style="padding: 20px; 
      background: white; 
      margin-top: 10px;
      -webkit-border-radius: 5px;
      -moz-border-radius: 5px;
      border-radius: 5px;">
  <img src="/images/posts/airbrake-grails-exception.png" style="display: block;"/>
</div>

Now log into Airbrake.io and you should see your exception listed.

<div style="padding: 20px; 
      background: #DADADA; 
      margin-top: 10px;
      -webkit-border-radius: 5px;
      -moz-border-radius: 5px;
      border-radius: 5px;">
  <img src="/images/posts/airbrake-airbrake-exception-list.png" style="display: block;"/>
  <img src="/images/posts/airbrake-airbrake-exception.png" style="display: block;"/>
</div>

As you can see, not only is the stacktrace available, but also the request parameters and the session data.

I wanted to thank [Phuong LeCong](https://github.com/plecong/grails-airbrake) for his hard work on this. I also appreciate the Grails community, especially the core contributors for the hard work they do to keep us all happy developers.

