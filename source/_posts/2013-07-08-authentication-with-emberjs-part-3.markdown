---
layout: post
title: "Authentication with EmberJS - Part 3"
date: 2013-07-08 19:39
comments: true
categories: 
- EmberJS
---

If you have not yet gone through [Part 1](/blog/2013/07/08/authentication-with-emberjs-part-1/) and [Part 2](/blog/2013/07/08/authentication-with-emberjs-part-2/), I recommend you do. You can check out the code up to this point with the following:

    $ git clone https://github.com/cavneb/simple_auth.git
    $ cd simple_auth
    $ git checkout step2
    $ bundle install
    $ rake db:migrate; rake db:migrate RAILS_ENV=test
    $ rake test

Also, make sure you run `./bin/ember_build` in a separate tab.

## What's Left?

So far, we have created an Ember application with a RailsAPI backend and can register, login and logout. There are a few more things that we want to be able to do before we can call it a wrap on this series.

1. Pass the access token with each request to the backend and require authorization for some data to return.
2. Force the user to the login page when they try to access a page which requires authentication.
3. Add validation to our registration form.

## Access Token In Each Request

Believe it or not, this is already happening. If you look at *auth_manager.js*, you will see that in the *authenticate* function, we add the headers to each AJAX request with the access token.

Let's test this.

Open up the *top_secret* route and load place the user list into the controllers model:

```javascript public/javascripts/routes/top_secret_route.js
var User = require('../models/user');

var TopSecretRoute = Ember.Route.extend({
  model: function() {
    return User.find();
  }
});

module.exports = TopSecretRoute;
```

Now update the *top_secret* template with the following:

{% raw %}
```html public/javascripts/templates/top_secret.hbs
<h2>Users (Top Secret Stuff)</h2>

<table class="table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Username</th>
    </tr>
  </thead>
  <tbody>
  {{#each controller}}
    <tr>
      <td>{{name}}</td>
      <td>{{email}}</td>
      <td>{{username}}</td>
    </tr>
  {{/each}}
  </tbody>
</table>
```
{% endraw %}

Refresh the browser and click on the *Top Secret* nav item. **If you haven't already registered and/or logged in, do it first.**

{% img /images/posts/simple-auth-ss-6.png %}

If you view the console when loading this page, you will see the network request made to */users*. In this request, you can see the headers sent out, one of which is the *Authorization* header. If this wasn't there, we would not be able to see a list of users. Click 'Logout' and then go to the Top Secret page again. See? You get a **401 Unauthorized** response from */users*.

<div style="background-color: #FDF6E3; padding: 10px; margin-bottom: 10px;">If you still see the users list, it is because they are cached. Refresh the page.</div>

## Hide Pages Which Require Authorization

It seems rather silly for us to be able to click on the *Top Secret* nav item and see an empty list of users. Let's require the user to be authenticated in order to view that page.

The easiest way to do this is to create a base route which can be extended by routes which require authentication.  Create a new route called *authenticated*.

    $ ember generate -r authenticated

```javascript public/javascripts/routes/authenticated_route.js
var AuthenticatedRoute = Ember.Route.extend({
  beforeModel: function(transition) {
    if (!App.AuthManager.isAuthenticated()) {
      this.redirectToLogin(transition);
    }
  },

  // Redirect to the login page and store the current transition so we can
  // run it again after login
  redirectToLogin: function(transition) {
    var sessionNewController = this.controllerFor('sessions.new');
    sessionNewController.set('attemptedTransition', transition);
    this.transitionTo('sessions.new');
  },

  events: {
    error: function(reason, transition) {
      this.redirectToLogin(transition);
    }
  }
});

module.exports = AuthenticatedRoute;
```

Now modify the *sessions/new* controller and redirect to the *attemptedTransition* if available:

```javascript public/javascripts/controllers/sessions/new_controller.js
var SessionsNewController = Ember.ObjectController.extend({

  attemptedTransition: null,

  loginUser: function() {
    var self = this;
    var router = this.get('target');
    var data = this.getProperties('username_or_email', 'password');
    var attemptedTrans = this.get('attemptedTransition');

    $.post('/session', data, function(results) {
      App.AuthManager.authenticate(results.api_key.access_token, results.api_key.user_id);
      if (attemptedTrans) {
        attemptedTrans.retry();
        self.set('attemptedTransition', null);
      } else {
        router.transitionTo('index');
      }
    });
  }
});

module.exports = SessionsNewController;
```

Finally, update the *top_secret* route to extend the new AuthenticatedRoute:

```javascript public/javascripts/routes/top_secret_route.js
var AuthenticatedRoute = require('./authenticated_route');
var User = require('../models/user');

var TopSecretRoute = AuthenticatedRoute.extend({
  model: function() {
    return User.find();
  }
});

module.exports = TopSecretRoute;
```

Refresh your browser and click on the *Top Secret* nav item. You should be redirected to the login page. Now log in and it should redirect you right back to the top secret page.

## Form Validation

The final thing I am going to cover (briefly) is performing form validation. This is very simple considering our backend is already giving us what we need. Open up the *user.js* model and add 'errors' as an attribute:

```javascript public/javascripts/models/user.js
var User = DS.Model.extend({
  name:     DS.attr('string'),
  email:    DS.attr('string'),
  username: DS.attr('string'),

  errors: {}
});

module.exports = User;
```

Now open up the *users/new* controller and capture the errors on a failed registration and place them into the errors hash:

```javascript public/javascripts/controllers/users/new_controller.js
var UsersNewController = Ember.ObjectController.extend({
  createUser: function() {
    var router = this.get('target');
    var data = this.getProperties('name', 'email', 'username', 'password', 'password_confirmation')
    var user = this.get('model');

    $.post('/users', { user: data }, function(results) {
      App.AuthManager.authenticate(results.api_key.access_token, results.api_key.user_id);
      router.transitionTo('index');
      
    }).fail(function(jqxhr, textStatus, error ) {
      if (jqxhr.status === 422) {
        errs = JSON.parse(jqxhr.responseText)
        user.set('errors', errs.errors);
      }
    });
  }
});

module.exports = UsersNewController;
```

Finally, update the registration template:

{% raw %}
```html public/javascripts/templates/users/new.hbs
<h2>Register</h2>

<form {{action "createUser" on="submit"}}>
  <div {{bindAttr class=":controls errors.name:error"}}>
    <label>Full Name</label>
    {{input type="text" value=name placeholder="Full Name"}}
    <small class="below">{{errors.name}}</small>
  </div>

  <div {{bindAttr class=":controls errors.email:error"}}>
    <label>Email Address</label>
    {{input type="email" value=email placeholder="Email Address"}}
    <small class="below">{{errors.email}}</small>
  </div>

  <div {{bindAttr class=":controls errors.username:error"}}>
    <label>Username</label>
    {{input type="text" value=username placeholder="Username"}}
    <small class="below">{{errors.username}}</small>
  </div>

  <div {{bindAttr class=":controls errors.password:error"}}>
    <label>Password</label>
    {{input type="password" value=password placeholder="Password"}}
    <small class="below">{{errors.password}}</small>
  </div>

  <div {{bindAttr class=":controls errors.password_confirmation:error"}}>
    <label>Confirm Password</label>
    {{input type="password" value=password_confirmation placeholder="Confirm Password"}}
    <small class="below">{{errors.password_confirmation}}</small>
  </div>

  <br>
  <button type="submit">Submit</button>
</form>
```
{% endraw %}

Refresh the browser and play around with the form. You should see error messages on submit if they are invalid. These errors are provided by Rails on the backend and are returned via the API.

{% img /images/posts/simple-auth-ss-7.png %}

## Done!

Thanks for sticking through this with me. The final code can be found at [https://github.com/cavneb/simple_auth](https://github.com/cavneb/simple_auth).

I especially want to thank all those who are taking the time to teach Ember via blogs, screencasts and live presentations. I find myself struggling less and less every day because new content comes out from all of you. Thanks!
