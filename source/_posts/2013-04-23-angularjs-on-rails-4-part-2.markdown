---
layout: post
title: "AngularJS on Rails 4 - Part 2"
date: 2013-04-23 07:35
comments: true
categories: 
---

Let's pick up where we left off. If you haven't already, make sure you go through [Part 1](/blog/2013/04/22/angularjs-on-rails-4-part-1/) to create your base Rails app with the API setup.

You can either continue using the code you have created on part 1 or you can catch up by checking out the tagged code:

    $ git clone https://github.com/cavneb/angular_casts
    $ cd angular_casts
    $ git checkout 'step-2'
    $ bundle install
    $ rake db:migrate

## Add AngularJS Libraries

There are a couple of different ways we can add AngularJS into our application. [Ryan Bates](http://railscasts.com/episodes/405-angularjs) suggests using the [angular-rails gem](https://github.com/hiravgandhi/angularjs-rails). I found this to be a bit outdated and overkill for what we are trying to do. Not only that, it's good to know how to do this without a gem.

Go to [http://code.angularjs.org/1.0.6/](http://code.angularjs.org/1.0.6/) and download the following files into your vendor/assets/javascripts folder:

* [http://code.angularjs.org/1.0.6/angular.js](http://code.angularjs.org/1.0.6/angular.js)
* [http://code.angularjs.org/1.0.6/angular-resource.js](http://code.angularjs.org/1.0.6/angular-resource.js)

Here's a shortcut to do this:

    $ wget http://code.angularjs.org/1.0.6/angular.js -P vendor/assets/javascripts
    $ wget http://code.angularjs.org/1.0.6/angular-resource.js -P vendor/assets/javascripts

### Setup the Javascript Folders

We want to keep our code organized. We will place our controllers, filters, services, directives, etc. in the app/assets/javascripts folder. Create the following directories:

* app/assets/javascripts/angular/controllers
* app/assets/javascripts/angular/directives
* app/assets/javascripts/angular/filters
* app/assets/javascripts/angular/models
* app/assets/javascripts/angular/services

Here's a shortcut to do this:

    $ mkdir -p app/assets/javascripts/angular/controllers \
               app/assets/javascripts/angular/directives \
               app/assets/javascripts/angular/filters \
               app/assets/javascripts/angular/models \
               app/assets/javascripts/angular/services

We will not use all of these folders, but it is good to have them there so when we do end up needing to create additional files, we know where to put them.    

### Update the Asset Pipeline 

Let's do some cleanup of the Asset Pipeline. First, create the main javascript file which will drive our AngularJS application. 

```coffeescript
# app/assets/javascripts/angular_casts.js.coffee

@app = angular.module('AngularCasts', ['ngResource'])
```

In this file we create a new module called *AngularCasts* and assign it to `@app`. We also add the dependency of `ngResource` which is our angular-resource.js file.

Now open up *app/assets/javascripts/application.js* and make the following changes:

```javascript
//= require angular
//= require angular-resource
//= require angular_casts
//= require_tree ./angular
```

The order of these is important due to the latter ones depending on the ones prior to them. Note that we do not have jQuery in here. AngularJS works great without jQuery and by removing it, we are forced to think the *Angular* way.

Let's now clean up our Gemfile. We no longer have any dependencies on *jquery*, *turbolinks* or *jbuilder*. Remove them from the Gemfile.

```ruby
# Gemfile

source 'https://rubygems.org'

gem 'rails', '4.0.0.beta1'
gem 'sqlite3'
gem 'feedzirra'

group :assets do
  gem 'sass-rails',   '~> 4.0.0.beta1'
  gem 'coffee-rails', '~> 4.0.0.beta1'
  gem 'uglifier', '>= 1.0.3'
end
```

Run `bundle install` to remove the gems from `Gemfile.lock`.

    $ bundle install

Now that we have removed this, we need to update our layout to not include these libraries. Open up *app/views/layouts/application.html.erb* and update the javascript and stylesheet includes to the following:

```ruby
<%= stylesheet_link_tag    "application", media: "all" %>
<%= javascript_include_tag "application" %>
```

## Let's Get It Working!

We now have everything in place for our AngularJS application except for the view. We are going to use a simple controller named `home` with a single view, `index`.

Run the following command:

    $ rails g controller home index

We now have a controller and view. Let's set this to as our *root_path* in `routes.rb`:

```ruby
# config/routes.rb

AngularCasts::Application.routes.draw do
  get '/episodes' => 'episodes#index', format: 'json'
  get '/episodes/:id' => 'episodes#show', format: 'json'

  root to: 'home#index'
end
```

*Note that the line `get 'home#index'` was removed. This is not needed because the root path directs to it.*

Let's start our app up and make sure it's working.

    $ rails s

Now open [http://localhost:3000](http://localhost:3000). Works? YAY!

### Adding Angular to our Views

For our app to recognize that it is an Angular application, we need to add the ng-app attribute to the html tag on our layout.

```html
<!-- app/views/layouts/application.html.erb -->

<html ng-app="AngularCasts">
```

Let's create a controller that will be used to list out the episodes. Create a new coffeescript file at *app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee*

```coffeescript
# app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee

@EpisodesCtrl = @app.controller 'EpisodesCtrl', ["$scope", ($scope) ->
  $scope.message = "Angular Rocks!"
]
```

Let's update our controller view as well to display the message.

```html
<!-- app/views/home/index.html.erb -->

<div ng-controller="EpisodesCtrl">
  <h1>Message: {{message}}</h1>
</div>
```

Here we have bound the contents of the *div* to the controller *EpisodesCtrl*. Refresh the browser and you should see 'Message: Angular Rocks!'

## List the Episodes

Now that we have Angular talking to our views, let's integrate our episodes data. Remember that we can access the JSON data by calling [http://localhost:3000/episodes.json](http://localhost:3000/episodes.json).

Modify your controller to perform the ajax request and place the results into the `$scope`

```coffeescript
# app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee

@EpisodesCtrl = @app.controller 'EpisodesCtrl', ["$scope", "$http", ($scope, $http) ->
  $scope.episodes = []

  loadEpisodes = ->
    $http.get("/episodes.json").success((data, status, headers, config) ->
      angular.forEach data, (value) ->
        $scope.episodes.push value.episode
    )

  loadEpisodes()
]
```

*** DESCRIPTION TO FOLLOW

Let's update our view as well to list out the episodes in the scope.

```html
<!-- app/views/home/index.html.erb -->

<div id="episode-list-container" ng-controller="EpisodesCtrl">
  <ul>
    <li ng-repeat="episode in episodes">
      <h3>{{episode.title}} <small>({{episode.duration}})</small></h3>
    </li>
  </ul>
</div>
```

Try it out. If all worked well, you should see a list of episodes. Now lets add the much needed CSS to our application. Copy the following into *app/assets/stylesheets/episodes.css.scss*

```css
/* app/assets/stylesheets/episodes.css.scss */

body {
  font-size: 12px;
  font-family: Helvetica, sans-serif;
  background-color: #ddd;
  margin: 0px;
}

#episode-list-container {
  background-color: #fff;
  border-right: 1px solid #d0d0d0;
  max-height: 400px;
  overflow-y: auto;
  width: 30%;
  float: left;
  ul {
    margin: 0px;
    list-style: none;
    padding: 0px;
    li {
      cursor: pointer;
      border-bottom: 1px solid #ddd;
      padding: 0 10px;
    }
  }
  h3 {
    font-size: 14px;
    small {
      font-size: 12px;
      color: #ccc;
      font-weight: normal;
    }
  }
}
#episode-view-container {
  float: left;
  width: 60%;
  margin: 15px;
  background-color: #fff;
  max-height: 400px;
  padding: 5px 15px 15px 15px;
}
```

Refresh the browser again. Ooooh!


















