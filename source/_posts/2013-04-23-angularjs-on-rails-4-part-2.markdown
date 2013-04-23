---
layout: post
title: "AngularJS on Rails 4 - Part 2"
date: 2013-04-23 07:35
comments: true
categories: 
- Rails
- AngularJS
- JavaScript
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

{% raw %}
```html
<!-- app/views/home/index.html.erb -->

<header>
  AngularCasts
</header>

<div ng-controller="EpisodesCtrl">
  <h1>Message: {{message}}</h1>
</div>
```
{% endraw %}

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

{% raw %}
```html
<!-- app/views/home/index.html.erb -->

<header>
  AngularCasts
</header>

<div ng-controller="EpisodesCtrl">
  <div id="episode-list-container" >
    <ul>
      <li ng-repeat="episode in episodes">
        <h3>{{episode.title}} <small>({{episode.duration}})</small></h3>
      </li>
    </ul>
  </div>
</div>
```
{% endraw %}

Try it out. If all worked well, you should see a list of episodes. Now lets add the much needed CSS to our application. Copy the following into *app/assets/stylesheets/episodes.css.scss*

```css
/* app/assets/stylesheets/episodes.css.scss */

body {
  font-size: 12px;
  font-family: Helvetica, sans-serif;
  background-color: #ddd;
  margin: 0px;
}

header {
  background-color: #4F4F4F;
  color: #fff;
  position: absolute;
  height: 36px;
  top: 0;
  left: 0;
  right: 0;
  font-size: 18px;
  line-height: 36px;
  font-weight: bold;
  padding-left: 15px;
}

#episode-list-container {
  background-color: #fff;
  position: absolute;
  width: 300px;
  top: 37px;
  left: 0;
  bottom: 0;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
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
    &.active {
      color: red;
    }
  }
}
#episode-view-container {
  position: absolute;
  border-left: 1px solid #d0d0d0;
  top: 37px;
  left: 300px;
  right: 0;
  bottom: 0;
  background-color: #fff;
  min-height: 400px;
  padding: 5px 25px;

  #player {
    border: 1px solid #000;
    max-width: 800px;
  }
}
```

Refresh the browser again. Ooooh!

## Show the Selected Episode

When we click on the episode on the left, we want to be able to view the episode with more information in the main area. This can be done in a few steps:

### Add an event listener to the episode

This is an easy part. Add the ng-click attribute to the *h3* tag to call the method *showEpisode*.

{% raw %}
```html
<h3 ng-click="showEpisode(episode)">{{episode.title}} <small>({{episode.duration}})</small></h3>
```
{% endraw %}

As you can see, we added `ng-click="showEpisode(episode)"` to the *h3* tag. The function call takes the episode as it's parameter, which is available from the ng-repeat. 

### Add the function into the controller

Let's add the *showEpisode* function to our controller.

```coffeescript
# app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee

@EpisodesCtrl = @app.controller 'EpisodesCtrl', ["$scope", "$http", ($scope, $http) ->

  $scope.episodes = []
  $scope.selectedEpisode = null

  $scope.showEpisode = (episode) ->
    $scope.selectedEpisode = episode

  loadEpisodes = ->
    $http.get("/episodes.json").success((data, status, headers, config) ->
      angular.forEach data, (value) ->
        $scope.episodes.push value.episode
    )

  loadEpisodes()
]
```

On line 6, we added the scoped variable *selectedEpisode* with a default value of *null*.

On lines 8-9, we added the scoped function which is called by the *ng-click* and sets the *selectedEpisode* value to the passed in episode.

Now back on the view, let's add the container for the *show* view.

{% raw %}
```html
<!-- app/views/home/index.html.erb -->

<header>
  AngularCasts
</header>

<div ng-controller="EpisodesCtrl">
  <div id="episode-list-container" >
    <ul>
      <li ng-repeat="episode in episodes">
        <h3 ng-click="showEpisode(episode)">{{episode.title}} <small>({{episode.duration}})</small></h3>
      </li>
    </ul>
  </div>

  <div id="episode-view-container" ng-show="selectedEpisode">
    <h2>{{selectedEpisode.title}}</h2>
    <p>{{selectedEpisode.description}}</p>

    <div id="player"></div>
  </div>
</div>
```
{% endraw %}

Here we have added lines 16-21. In our container div, there is the attribute `ng-show="selectedEpisode"`. This ensures that the div is only visible if the scoped variable *selectedEpisode* is set. Line 20 will contain our video, but we'll get to that.

Then inside the div, we display the episode information for the selected episode.

Try it out by [reloading your browser](http://localhost:3000).

### Embed the Video

Now that we can see the individual episode information, we can add the video embed code. For this we are going to use [flowplayer](http://flowplayer.org). It is fairly simple to set up.

First we need to add the required *css* and *javascript* libraries to our layout. Open up *application.html.erb* and make the following changes:

```html
<!-- app/views/layouts/application.html.erb -->

<!DOCTYPE html>
<html ng-app="AngularCasts">
<head>
  <title>AngularCasts</title>
  <%= stylesheet_link_tag "application", "http://releases.flowplayer.org/5.4.0/skin/minimalist.css" %>
  <%= javascript_include_tag "application",
                             "http://ajax.googleapis.com/ajax/libs/jquery/1.8/jquery.min.js",
                             "http://releases.flowplayer.org/5.4.0/flowplayer.min.js" %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

Here we added a new stylesheet and two additional javascript files to our include tags. As you can see, jQuery was included. This is a dependency of flowplayer, so we had to add it. But we have made it this far without it!

Now we can embed the video from within our controller. This is very simple.

```coffeescript
# app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee

@EpisodesCtrl = @app.controller 'EpisodesCtrl', ["$scope", "$http", ($scope, $http) ->

  $scope.episodes = []
  $scope.selectedEpisode = null

  $scope.showEpisode = (episode) ->
    $scope.selectedEpisode = episode
    loadVideo(episode)

  loadVideo = (episode) ->
    $("#player").flowplayer
      playlist: [[mp4: episode.video_url]]
      ratio: 9 / 14

  loadEpisodes = ->
    $http.get("/episodes.json").success((data, status, headers, config) ->
      angular.forEach data, (value) ->
        $scope.episodes.push value.episode
    )

  loadEpisodes()
]
```

On line 10, we added a call to the function *loadVideo* passing in the episode.

On lines 12-15, we define a function which performs the javascript in order to load the video into `<div id="player"></div>`. You can see that we pass in the episode's video_url as the mp4 and set the ratio to 9/14.

Let's give it another go. Refresh your browser. You should now have a fully-functional AngularJS application!!!

## Visual Feedback

The final item we will be adding in our application is some visual feedback as to which video we are watching. In other words, we want to highlight the selected video on the left list.

Add the following *ng-class* attribute to the *h3* tag:

```html
...
<li ng-repeat="episode in episodes">
  <h3 ng-click="showEpisode(episode)" ng-class="isSelected(episode)">{{episode.title}} <small>({{episode.duration}})</small></h3>
</li>
...
```

We now are using the function `isSelected(episode)` as the class name. Let's add this to our controller.

```coffeescript
# app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee

@EpisodesCtrl = @app.controller 'EpisodesCtrl', ["$scope", "$http", ($scope, $http) ->

  $scope.episodes = []
  $scope.selectedEpisode = null

  $scope.showEpisode = (episode) ->
    $scope.selectedEpisode = episode
    loadVideo(episode)

  $scope.isSelected = (episode) ->
    'active' if $scope.selectedEpisode == episode

  loadVideo = (episode) ->
    $("#player").flowplayer
      playlist: [[mp4: episode.video_url]]
      ratio: 9 / 14

  loadEpisodes = ->
    $http.get("/episodes.json").success((data, status, headers, config) ->
      angular.forEach data, (value) ->
        $scope.episodes.push value.episode
    )

  loadEpisodes()
]
```

On lines 12-13, we have added the function *isSelected*. It will return 'active' if the selectedEpisode is the episode that was clicked.

## Final Product

{% img http://f.cl.ly/items/2r0I1w1L3d1s0N0l2A0v/angular_casts.png %}

