---
layout: post
title: "Authentication with EmberJS - Part 2"
date: 2013-07-08 17:36
comments: true
categories: 
- EmberJS
- Rails
- Rails::API
---

If you have not yet gone through [Part 1](/blog/2013/07/08/authentication-with-emberjs-part-1/), I recommend you do. You can check out the code up to this point with the following:

    $ git clone https://github.com/cavneb/simple-auth.git simple_auth
    $ cd simple_auth
    $ git checkout part-1-completed
    $ bundle install
    $ rake db:migrate; rake db:migrate RAILS_ENV=test
    $ rake test

## Add Ember using Ember Tools!

I have created Ember applications using a variety of shortcuts ([Yeoman](https://github.com/yeoman/generator-ember), [ember-rails](https://github.com/emberjs/ember-rails)) but have found that [Ember Tools](https://github.com/rpflorence/ember-tools) is by far the best option available. It allows me to skip the [Asset Pipeline](http://coderberry.me/blog/2012/04/24/asset-pipeline-for-dummies/) completely and work directly in my public folder.

To get started, install Ember Tools using *npm*.

    $ npm install -g ember-tools

Once this is installed, you will be able to use the console command *ember*. Try it out:

    $ ember -V
    0.2.4

Excellent. Now create our Ember app in our public directory with the following command:

    $ ember create --js-path public/javascripts
       skipped: .
       created: ./public/javascripts
       created: ./public/javascripts/vendor
       created: ./public/javascripts/config
       created: ./public/javascripts/controllers
       created: ./public/javascripts/helpers
       created: ./public/javascripts/models
       created: ./public/javascripts/routes
       created: ./public/javascripts/templates
       created: ./public/javascripts/views
       created: ./public/javascripts/mixins
       created: ./ember.json
       created: ./public/javascripts/config/app.js
       created: ./public/javascripts/config/store.js
       created: ./public/javascripts/config/routes.js
       created: ./public/javascripts/templates/application.hbs
       created: ./public/javascripts/templates/index.hbs
       created: ./index.html
       created: ./public/javascripts/vendor/ember-data.js
       created: ./public/javascripts/vendor/ember.js
       created: ./public/javascripts/vendor/handlebars.js
       created: ./public/javascripts/vendor/jquery.js
       created: ./public/javascripts/vendor/localstorage_adapter.js
    All done! Start with `config/routes.js` to add routes to your app.

With that simple command we now have a *nearly* functional Ember application. Let's move the generated *index.html* file into the public folder and modify it a tiny bit.

    $ mv index.html public/.

```html public/index.html
<!doctype html>
<html lang="en">
<head>
  <meta http-equiv="Content-type" content="text/html; charset=utf-8">
  <title>Ember App</title>
</head>
<body>
  <script src="javascripts/application.js"></script>
</body>
</html>
```

Note that the only thing that changed in this file is the path to the application.js file. Go ahead and start up your Rails application and visit [http://localhost:3000](http://localhost:3000).

    $ rails s

You shouldn't see anything come up and will likely see an error in the server logs. This is because the page is trying to load application.js when it does not exist. To create the file, run (in another terminal tab within the same root directory):

    $ ember build
       created: public/javascripts/templates.js
       created: public/javascripts/index.js
       created: public/javascripts/application.js
    build time: 358 ms

This created three files: templates.js, index.js and application.js. The two former are used temporarily to create the latter. Now refresh your browser and you should see the starter app:

{% img /images/posts/simple-auth-ss-1.png %}

Running *ember build* can get very tedious, so let's create a script which will monitor the file structure and run the command when needed. You will need to have *fsmonitor* installed if you don't already:

    $ npm install -g fsmonitor

Create the file bin/ember_build:

```bash bin/ember_build.sh
#!/bin/bash
fsmonitor -p -d public/javascripts '!index.js' '!templates.js' '!application.js' ember build -d
```

Now in a separate tab, make the file executable and run it:

    $ chmod a+x bin/ember_build.sh
    $ ./bin/ember_build.sh

    Monitoring:  public/javascripts
        filter:  **/ !**/index.js/** !**/templates.js/** !**/application.js/**
        action:  ember build

    ...

Now whenever we change our Ember app, the code will re-compile.

## Generate, Generate, Generate!

Ember Tools comes with generators, which I LOVE! Let's create some files using the generators and fill out our layout page.

Start by creating the route, handlebars template and **object** controller for *users/new*. This will be where we register.

    $ ember generate -rtc users/new
    -> What kind of controller: object, array, or neither? [o|a|n]: o
       created: public/javascripts/controllers/users/new_controller.js
       created: public/javascripts/templates/users/new.hbs
       created: public/javascripts/routes/users/new_route.js

Now create the route, handlebars template and **object** controller for *sessions/new*. This will be where we login.

    $ ember generate -rtc sessions/new
    -> What kind of controller: object, array, or neither? [o|a|n]: o
       created: public/javascripts/controllers/sessions/new_controller.js
       created: public/javascripts/templates/sessions/new.hbs
       created: public/javascripts/routes/sessions/new_route.js

Finally, create a page which is **TOP SECRET** and will require authentication to access. Let's use an **array** controller so we can list the users.

    $ ember generate -rtc top_secret
    -> What kind of controller: object, array, or neither? [o|a|n]: a
       created: public/javascripts/controllers/top_secret_controller.js
       created: public/javascripts/templates/top_secret.hbs
       created: public/javascripts/routes/top_secret_route.js

Update the application handlebars template to show links to the different pages.

{% raw %}
```html public/javascripts/templates/application.hbs
<div class="container">
  <div class="navbar">
    <div class="navbar-inner">
      <a class="brand" href="#">Simple Auth</a>
      <ul class="nav">
        <li>{{#linkTo 'index'}}Home{{/linkTo}}</li>
        <li>{{#linkTo 'top_secret'}}Top Secret{{/linkTo}}</li>
        <li>{{#linkTo 'users.new'}}Register{{/linkTo}}</li>
        <li>{{#linkTo 'sessions.new'}}Login{{/linkTo}}</li>
      </ul>
    </div>
  </div>

  {{outlet}}
</div>
```
{% endraw %}

Before these links will work we need to add the routes to the config/routes.js file:

```javascript public/javascripts/config/routes.js
var App = require('./app');

App.Router.map(function() {
  this.resource('sessions', function() {
    this.route('new');
  });
  this.resource('users', function() {
    this.route('new');
  })
  this.route('top_secret');
});
```

Refresh the browser and you should see something like this:

{% img /images/posts/simple-auth-ss-2.png %}

Add some style with [twitter bootstrap](http://www.bootstrapcdn.com/) by adding the CSS link in your *index.html* page:

```html public/index.html
<head>
  ...
  <link href="//netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.no-icons.min.css" rel="stylesheet">
</head>
```

Refresh. You can click on the links as well and you should see the correct pages load.

{% img /images/posts/simple-auth-ss-3.png %}

## Update to the Latest Ember Data

At the moment, Ember Tools does not provide the [latest version](http://builds.emberjs.com.s3.amazonaws.com/ember-data-latest.js) of Ember Data, so we will need to add this manually. Save the following file to the path *public/javascripts/vendor*:

    $ wget -P public/javascripts/vendor/ http://builds.emberjs.com.s3.amazonaws.com/ember-data-latest.js

Now update a your main application config file to make sure we are using the latest:

```javascript public/javascripts/config/app.js
require('../vendor/jquery');
require('../vendor/handlebars');
require('../vendor/ember');
require('../vendor/ember-data-latest');

var App = window.App = Ember.Application.create();
App.Store = require('./store');

module.exports = App;
```

## Auth Manager

On the blog post found at * [http://log.simplabs.com/post/53016599611/authentication-in-ember-js](http://log.simplabs.com/post/53016599611/authentication-in-ember-js)
, Marco Otte-Witte ([@simplabs](https://twitter.com/simplabs)) created a simple **AuthManager** which stores and handles authentication. It is very elegant and once I found this post, I got very excited. I made some minor tweaks to the code, but it is still largely intact.

Create a file in your *public/javascripts/config* folder called *auth_manager.js*:

```javascript public/javascripts/config/auth_manager.js
var User = require('../models/user');

var AuthManager = Ember.Object.extend({

  // Load the current user if the cookies exist and is valid
  init: function() {
    this._super();
    var accessToken = $.cookie('access_token');
    var authUserId  = $.cookie('auth_user');
    if (!Ember.isEmpty(accessToken) && !Ember.isEmpty(authUserId)) {
      this.authenticate(accessToken, authUserId);
    }
  },

  // Determine if the user is currently authenticated.
  isAuthenticated: function() {
    return !Ember.isEmpty(this.get('apiKey.accessToken')) && !Ember.isEmpty(this.get('apiKey.user'));
  },

  // Authenticate the user. Once they are authenticated, set the access token to be submitted with all
  // future AJAX requests to the server.
  authenticate: function(accessToken, userId) {
    $.ajaxSetup({
      headers: { 'Authorization': 'Bearer ' + accessToken }
    });
    var user = User.find(userId);
    this.set('apiKey', App.ApiKey.create({
      accessToken: accessToken,
      user: user
    }));
  },

  // Log out the user
  reset: function() {
    App.__container__.lookup("route:application").transitionTo('sessions.new');
    Ember.run.sync();
    Ember.run.next(this, function(){
      this.set('apiKey', null);
      $.ajaxSetup({
        headers: { 'Authorization': 'Bearer none' }
      });
    });
  },

  // Ensure that when the apiKey changes, we store the data in cookies in order for us to load
  // the user when the browser is refreshed.
  apiKeyObserver: function() {
    if (Ember.isEmpty(this.get('apiKey'))) {
      $.removeCookie('access_token');
      $.removeCookie('auth_user');
    } else {
      $.cookie('access_token', this.get('apiKey.accessToken'));
      $.cookie('auth_user', this.get('apiKey.user.id'));
    }
  }.observes('apiKey')
});

// Reset the authentication if any ember data request returns a 401 unauthorized error
DS.rejectionHandler = function(reason) {
  if (reason.status === 401) {
    App.AuthManager.reset();
  }
  throw reason;
};

module.exports = AuthManager;
```

For this to work, we will need to add include jquery.cookies into our app. Download [https://raw.github.com/carhartl/jquery-cookie/master/jquery.cookie.js](https://raw.github.com/carhartl/jquery-cookie/master/jquery.cookie.js) into the folder *public/javascripts/vendor* and update the *app.js* file:

    $ wget -P public/javascripts/vendor/ https://raw.github.com/carhartl/jquery-cookie/master/jquery.cookie.js

```javascript public/javascripts/config/app.js
require('../vendor/jquery');
require('../vendor/jquery.cookie');
require('../vendor/handlebars');
require('../vendor/ember');
require('../vendor/ember-data-latest');

var App = window.App = Ember.Application.create();
App.Store = require('./store');

module.exports = App;
```

<div style="background-color: #FDF6E3; padding: 10px; margin-bottom: 10px;">Note: You may have to do what I did on line 7 above by adding setting the application to window.App as well. If you have troubles, this is likely why.</div>

Now create the application router and add the AuthManager to the App in the *init* function. The reason it goes here is because it's the first thing that gets run after all the code has been loaded.

    $ ember generate -r application

```javascript public/javascripts/routes/application_route.js
var AuthManager = require('../config/auth_manager');

var ApplicationRoute = Ember.Route.extend({
  init: function() {
    this._super();
    App.AuthManager = AuthManager.create();
  }
});

module.exports = ApplicationRoute;
```

## Registration

Let's create the parts of our app which will allow a user to register. We want to start off by creating a user model which uses Ember Data:

    $ ember generate -m user

```javascript public/javascripts/models/user.js
var User = DS.Model.extend({
  name:     DS.attr('string'),
  email:    DS.attr('string'),
  username: DS.attr('string')
});

module.exports = User;
```

While we're here, let's also create the model for *api_key*:

    $ ember generate -m api_key

```javascript public/javascripts/models/api_key.js
// Ember.Object instead of DS.Model because this will never persist to or query the server
var ApiKey = Ember.Object.extend({
  access_token: '',
  user: null
});

module.exports = ApiKey;
```

<div style="background-color: #FDF6E3; padding: 10px; margin-bottom: 10px;"><strong>Important:</strong> I changed the type of object for ApiKey from <em>DS.Model</em> to <em>Ember.Object</em>. I did this because we will never persist to or query the server for API keys.</div>

For us to use Ember Data, we need to enable it. By default with Ember Tools, the localstorage adapter is enabled by default. Let's remove that and set the adapter to the REST adapter. Open up *config/store.js* and make the following changes:

```javascript public/javascripts/config/store.js
module.exports = DS.Store.extend({
  adapter: DS.RESTAdapter.create()
});
```

Open up our route for new users and set the model to be a new User record:

```javascript public/javascripts/routes/users/new_route.js
var User = require('../../models/user');

var UsersNewRoute = Ember.Route.extend({
  setupController: function(controller, model) {
    this.controller.set('model', User.createRecord());
  }
});

module.exports = UsersNewRoute;
```

Modify the users/new controller with the following:

```javascript public/javascripts/controllers/users/new_controller.js
var UsersNewController = Ember.ObjectController.extend({
  createUser: function() {
    var router = this.get('target');
    var data = this.getProperties('name', 'email', 'username', 'password', 'password_confirmation')
    var user = this.get('model');

    $.post('/users', { user: data }, function(results) {
      App.AuthManager.authenticate(results.api_key.access_token, results.api_key.user_id);
      router.transitionTo('index');
    });
  }
});

module.exports = UsersNewController;
```

Now let's update the handlebars template to show the registration form:

{% raw %}
```html public/javascripts/templates/users/new.hbs
<h2>Register</h2>

<form {{action "createUser" on="submit"}}>
  <div>
    <label>Full Name</label>
    {{input type="text" value=name placeholder="Full Name"}}
  </div>

  <div>
    <label>Email Address</label>
    {{input type="email" value=email placeholder="Email Address"}}
  </div>

  <div>
    <label>Username</label>
    {{input type="text" value=username placeholder="Username"}}
  </div>

  <div>
    <label>Password</label>
    {{input type="password" value=password placeholder="Password"}}
  </div>

  <div>
    <label>Confirm Password</label>
    {{input type="password" value=password_confirmation placeholder="Confirm Password"}}
  </div>

  <br>
  <button type="submit">Submit</button>
</form>
```
{% endraw %}

Refresh your browser and fill out the registration form and hit submit. You should be logged in and redirected to the index page.

{% img /images/posts/simple-auth-ss-4.png %}

In your JavaScript console, you can view the currently logged in user with the following:

    > App.AuthManager.get('apiKey.user.name')
      "Eric Berry"
    > App.AuthManager.isAuthenticated()
      true

## Current User in Nav Bar

We're doing great. We now have created an account. However, the UI hasn't changed. We want to be told that we are logged in and be given the option to log out.

Let's create an *application* controller with some computed properties which we will use in the template:

    $ ember generate -c application
    -> What kind of controller: object, array, or neither? [o|a|n]: n
       created: public/javascripts/controllers/application_controller.js

```javascript public/javascripts/controllers/application_controller.js
var ApplicationController = Ember.Controller.extend({
  currentUser: function() {
    return App.AuthManager.get('apiKey.user')
  }.property('App.AuthManager.apiKey'),

  isAuthenticated: function() {
    return App.AuthManager.isAuthenticated()
  }.property('App.AuthManager.apiKey')
});

module.exports = ApplicationController;
```

Now modify the application handlebars template to show the menu based on whether the user is authenticated or not:

{% raw %}
```html public/javascripts/templates/application.hbs
<div class="container">
  <div class="navbar">
    <div class="navbar-inner">
      <a class="brand" href="#">Simple Auth</a>
      <ul class="nav">
        <li>{{#linkTo 'index'}}Home{{/linkTo}}</li>
        <li>{{#linkTo 'top_secret'}}Top Secret{{/linkTo}}</li>

        {{#if isAuthenticated}}
          <li><a href="#">{{currentUser.email}}</a></li>
          <li><a href="#" {{action 'logout'}}>Logout</a></li>
        {{else}}
          <li>{{#linkTo 'users.new'}}Register{{/linkTo}}</li>
          <li>{{#linkTo 'sessions.new'}}Login{{/linkTo}}</li>
        {{/if}}
      </ul>
    </div>
  </div>

  {{outlet}}
</div>
```
{% endraw %}

Now when we reload the browser it will show our email address when we are logged in with a link to log out. Try it out.

## Logout

We have an action set up in our application template to log out, but we don't have an event to handle it yet. Let's put this in the application route.

```javascript public/javascripts/routers/application_route.js
var AuthManager = require('../config/auth_manager');

var ApplicationRoute = Ember.Route.extend({
  init: function() {
    this._super();
    App.AuthManager = AuthManager.create();
  },

  events: {
    logout: function() {
      App.AuthManager.reset();
      this.transitionTo('index');
    }
  }
});

module.exports = ApplicationRoute;
```

Refresh your browser and click 'Logout'. Works? YAY!!!

## Login

Let's start by updating our the session/new route to assign an Ember Object as the controller's model:

```javascript public/javascripts/routes/sessions/new_route.js
var SessionsNewRoute = Ember.Route.extend({
  model: function() {
    return Ember.Object.create();
  }
});

module.exports = SessionsNewRoute;
```

Now update the sessions/new controller to perform the login:

```javascript public/javascripts/controllers/sessions/new_controller.js
var SessionsNewController = Ember.ObjectController.extend({
  loginUser: function() {
    var router = this.get('target');
    var data = this.getProperties('username_or_email', 'password');

    $.post('/session', data, function(results) {
      App.AuthManager.authenticate(results.api_key.access_token, results.api_key.user_id);
      router.transitionTo('index');
    });
  }
});

module.exports = SessionsNewController;
```

Finally, update the handlebars template to show the login form:

{% raw %}
```html public/javascripts/templates/sessions/new.hbs
<h2>Login</h2>

<form {{action "loginUser" on="submit"}}>
  <div>
    <label>Username or Email</label>
    {{input type="text" value=username_or_email placeholder="Username or Email Address"}}
  </div>

  <div>
    <label>Password</label>
    {{input type="password" value=password placeholder="Password"}}
  </div>

  <br>
  <button type="submit">Submit</button>
</form>
```
{% endraw %}

Refresh your browser and log in. On success, you should be redirected to the index page and the nav bar should indicate you are logged in.

{% img /images/posts/simple-auth-ss-5.png %}

<h3><a href="/blog/2013/07/08/authentication-with-emberjs-part-3/">Continue to Part 3</a></h3>