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

###### This article is still being worked on. I didn't expect to get the response I did and felt that it was OK that I posted it early for the use of the Utah Ruby User's Group. I will update the article within a couple of days.

Let's pick up where we left off. If you haven't already, make sure you go through [Part 1](/blog/2013/04/22/angularjs-on-rails-4-part-1/) to create your base Rails app with the API setup.

You can either continue using the code you have created on part 1 or you csan catch up by checking out the tagged code:

    $ git clone https://github.com/cavneb/angular_casts
    $ cd angular_casts
    $ git checkout step-1
    $ bundle install
    $ rake db:migrate; rake db:migrate RAILS_ENV=test
    $ rake test

## Add AngularJS Libraries

There are a couple of different ways we can add AngularJS into our application. [Ryan Bates](http://railscasts.com/episodes/405-angularjs) suggests using the [angular-rails gem](https://github.com/hiravgandhi/angularjs-rails). I found this to be a bit outdated and overkill for what we are trying to do. Not only that, it's good to know how to do this without a gem.

Go to [http://code.angularjs.org/1.0.6/](http://code.angularjs.org/1.0.6/) and download the following files into your vendor/assets/javascripts folder:

* [http://code.angularjs.org/1.0.6/angular.js](http://code.angularjs.org/1.0.6/angular.js)
* [http://code.angularjs.org/1.0.6/angular-resource.js](http://code.angularjs.org/1.0.6/angular-resource.js)
* [http://code.angularjs.org/1.0.6/angular-mocks.js](http://code.angularjs.org/1.0.6/angular-mocks.js) (this will be used for testing later)

Here's a shortcut to do this:

    $ wget http://code.angularjs.org/1.0.6/angular.js -P vendor/assets/javascripts
    $ wget http://code.angularjs.org/1.0.6/angular-resource.js -P vendor/assets/javascripts
    $ wget http://code.angularjs.org/1.0.6/angular-mocks.js -P vendor/assets/javascripts

###### We could always use a CDN instead of having a local version of these libraries. However, having them locally allows the the asset pipeline to minify and concatenate it with the other javascript files.

### Setup the Javascript Folders

We want to keep our code organized. We will place our controllers, filters, services, directives, etc. in the app/assets/javascripts folder. Create the following directories:

* app/assets/javascripts/angular/controllers
* app/assets/javascripts/angular/directives
* app/assets/javascripts/angular/services

Here's a shortcut to do this:

    $ mkdir -p app/assets/javascripts/angular/controllers \
               app/assets/javascripts/angular/directives \
               app/assets/javascripts/angular/services

Now let's create the main javascript file which will drive our AngularJS application.

``` coffeescript app/assets/javascripts/app.js.coffee
window.App = angular.module('AngularCasts', ['ngResource'])
```

In this file we create a new module called *AngularCasts* and assign it to `window.App`. We also add the dependency of `ngResource` which is our angular-resource.js file.

Next, we need to update our JavaScript manifest to include the our Angular scripts.

``` javascript app/assets/javascripts/application.js
//= require angular
//= require angular-resource
//= require app
//= require_tree ./angular
```

The order of these is important due to the latter ones depending on the ones prior to them.

## Add the View

We currently do not have any views setup. This is because rails-api doesn't create them by default. Not a worry.

Create a new folder where we will store our views. This will be in the *app/views* folder.

    $ mkdir -p app/views

Next, we need to create a controller. This will allow us to set up a route to a view.

    $ rails-api g controller home index

This set up the HomeController and added the action `index`. This command did not create our view file so we need to create that as well. Create the folder and file `app/views/home/index.html.erb` and add the following:

{% raw %}
``` html app/views/home/index.html.erb
<!doctype html>
<html ng-app>
  <head>
    <title>Angular Casts</title>
  </head>
  <body>
    <div>
      <label>Name:</label>
      <input type="text" ng-model="yourName" placeholder="Enter a name here">
      <hr>
      <h1>Hello {{yourName}}!</h1>
    </div>

    <%= javascript_include_tag "application" %>
  </body>
</html>
```
{% endraw %}

Update your routes file to use this view as the root.

``` ruby config/routes.rb
AngularCasts::Application.routes.draw do
  scope :api do
    get "/screencasts(.:format)" => "screencasts#index"
    get "/screencasts/:id(.:format)" => "screencasts#show"
  end
  root to: "home#index"
end
```

*Note that the line `get 'home#index'` was removed. This is not needed because the root path directs to it.*

On a regular Rails application, you typically have a layout for each view. Because this is a *single page application*, we don't need this. For our home page to load properly, make sure the layout is not rendered for the index view. This is completely optional and not a good way to go if you plan on adding additional views sharing the same layout.

``` ruby app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    render layout: false
  end
end
```

Start up your server and open up [http://localhost:3000](http://localhost:3000). Type in your name into the text field. If the content changes as you type, it worked! You now have a functional Angular application!

{% img https://dl-web.dropbox.com/get/Public/Assets/angular_casts_1.gif?w=AAAI1JnNBiWomAz4enNeeNU-MX0qPe0dAs0LmdJKJoXoug %}

###### If you are using Rails 3, you will need to delete the file *public/index.html*

## Now the fun begins!

You may have noticed that our view has the code `<html ng-app>`. This tells the page that it is an Angular application. In order for us to tell the page that it should use the *App* module, we need to add the module name to the *directive*. Remove the *ng-app* directive from the *html* tag and move it to the *body* tag with the app name set to *AngularCasts*:

``` html app/views/layouts/application.html.erb
<!doctype html>
<html>
  <head>
    <title>Angular Casts</title>
  </head>
  <body ng-app="AngularCasts">
    ...
```

Now our view knows to use the AngularCasts module.

## Create an Angular Controller

Let's create a controller that will be used to list out the episodes. Create a new coffeescript file at *app/assets/javascripts/angular/controllers/episodes_ctrl.js.coffee*

```coffeescript app/assets/javascripts/angular/controllers/screencasts_ctrl.js.coffee
window.App.controller 'ScreencastsCtrl', ["$scope", ($scope) ->
  $scope.message = "Angular Rocks!"
]
```

On line 1, we create a new Angular controller belonging to *App* named *ScreencastsCtrl*. The controller will be referenced in our view as *ScreencastCtrl*. For more information on Angular controllers, read [http://docs.angularjs.org/guide/dev_guide.mvc.understanding_controller](http://docs.angularjs.org/guide/dev_guide.mvc.understanding_controller).

Let's update our view to display the message.

{% raw %}
```html app/views/home/index.html.erb
<!doctype html>
<html>
  <head>
    <title>Angular Casts</title>
  </head>
  <body ng-app="AngularCasts">
    <header>
      Angular Casts
    </header>

    <div ng-controller="ScreencastsCtrl" id="screencast-ctrl">
      <h1>Message: {{message}}</h1>
    </div>

    <%= javascript_include_tag "application" %>
  </body>
</html>
```
{% endraw %}

Here we have bound the contents of the *div* to the controller *ScreencastsCtrl*. Refresh the browser and you should see 'Message: Angular Rocks!'.

It works, but it's ugly. Lets add the much needed CSS to our application. Copy the following into *app/assets/stylesheets/app.css.scss*. You will need to create the *stylesheets* folder first.

``` scss app/assets/stylesheets/app.css.scss
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

#screencast-ctrl {
  background-color: #fff;
  position: absolute;
  top: 37px;
  width: 100%;
  bottom: 0;
  overflow: auto;
}

#screencast-list-container {
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

#screencast-view-container {
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

We will also need to create a stylesheets manifest in order for the stylesheets to be a part of the asset pipeline. Create the manifest *application.css*:

``` css app/assets/stylesheets/application.css
/*
 *= require_self
 *= require app
 */
```

Now add the stylesheet include tag into the `<head>` of your view.

```html app/views/home/index.html.erb
<!doctype html>
<html>
  <head>
    <title>Angular Casts</title>
    <%= stylesheet_link_tag "application", :media => "all" %>
  </head>
  ...
```

Refresh the browser. Ooooh!

<h2>Testing with <del>Testacular</del> Karma</h2>

There wasn't a lot of good documentation on the web on how to test your AngularJS application on Rails, but I muddled through it. Hopefully, you will be able to take what I learned and not have to spend tons of time figuring it out on your own. Many thanks to [this post](http://blog.freeside.co/post/41774841006/getting-started-with-angular-unit-tests) and [this gist](https://gist.github.com/jrmoran/4163147)!!!

### Install Karma

[Karma](http://karma-runner.github.io/0.8/index.html) (formerly Testacular) is a simple tool that allows you to execute JavaScript code in multiple real browsers, powered by [Node.js](http://nodejs.org/) and [Socket.io](http://socket.io/). The main purpose of Karma is to make your TDD development easy, fast, and fun. *We'll just see about that!*

To install Karma, follow the installation instructions [found here](https://github.com/karma-runner/karma#npm-installation). Please do not leave comments on how to install Karma or NPM. This is not in the scope of this blog post and their websites would be much better suited to teach these things. 

If you already have NPM installed, you can simply run the following to install Karma:

    $ npm install -g karma

The instructions continue on about running the `karma init` command. I did that, but then had to do some modifications to the conf file that it generated. Here is my modified *karma.conf.js* file. Place this in the root directory of your app.

``` javascript karma.conf.js
// base path, that will be used to resolve files and exclude
basePath = './';

// list of files / patterns to load in the browser
files = [
  JASMINE,
  JASMINE_ADAPTER,

  // dependencies
  'vendor/assets/javascripts/angular.js', 
  'vendor/assets/javascripts/angular-resource.js', 
  'vendor/assets/javascripts/angular-mocks.js',

  // application
  'app/assets/javascripts/app.js.coffee', 
  'app/assets/javascripts/angular/**/*.js.coffee',

  // tests
  'test/**/*_spec.js',
  'test/**/*_spec.js.coffee'
];

// list of files to exclude
exclude = [];

// use dots reporter, as travis terminal does not support escaping sequences
// possible values: 'dots', 'progress', 'junit', 'teamcity'
// CLI --reporters progress
reporters = ['progress'];

// web server port
// CLI --port 9876
port = 9876;

// cli runner port
// CLI --runner-port 9100
runnerPort = 9100;

// enable / disable colors in the output (reporters and logs)
// CLI --colors --no-colors
colors = true;

// level of logging
// possible values: LOG_DISABLE || LOG_ERROR || LOG_WARN || LOG_INFO || LOG_DEBUG
// CLI --log-level debug
logLevel = LOG_INFO;

// enable / disable watching file and executing tests whenever any file changes
// CLI --auto-watch --no-auto-watch
autoWatch = true;

// Start these browsers, currently available:
// - Chrome
// - ChromeCanary
// - Firefox
// - Opera
// - Safari (only Mac)
// - PhantomJS
// - IE (only Windows)
// CLI --browsers Chrome,Firefox,Safari
browsers = ['Chrome'];

// If browser does not capture in given timeout [ms], kill it
// CLI --capture-timeout 5000
captureTimeout = 5000;

// Auto run tests on start (when browsers are captured) and exit
// CLI --single-run --no-single-run
singleRun = false;

// report which specs are slower than 500ms
// CLI --report-slower-than 500
reportSlowerThan = 500;

// compile coffee scripts
preprocessors = {
    '**/*.coffee': 'coffee'
}; 
```

Feel free to tweak the configurations. For all available config options and default values, see [https://github.com/karma-runner/karma/blob/stable/lib/config.js#L54](https://github.com/karma-runner/karma/blob/stable/lib/config.js#L54) *(not for the faint of heart)*.

That's all we need to get things ready to test. Go ahead and start the Karma engine:

    $ karma start

    INFO [karma]: Karma server started at http://localhost:9876/
    INFO [launcher]: Starting browser Chrome
    WARN [watcher]: Pattern "/Users/eberry/Personal/URUG/angular_casts/test/**/*_spec.js" does not match any file.
    WARN [watcher]: Pattern "/Users/eberry/Personal/URUG/angular_casts/test/**/*_spec.js.coffee" does not match any file.
    INFO [Chrome 26.0 (Mac)]: Connected on socket id 02fKE18DSIEh_-pMbjfz
    Chrome 26.0 (Mac): Executed 0 of 0 SUCCESS (0.059 secs / 0 secs)

If all worked well, a browser should pop up that looks something like this.

{% img https://photos-6.dropbox.com/t/0/AAA5QkzFN62Fb6Mt46oKocZitqWkC4s0TFTXMseswo3uIA/12/2176587/png/32x32/3/_/1/2/Screen%20Shot%202013-05-02%20at%209.32.00%20PM.png/vdIeOC67WkA2A2auFY6K7iCUWoEwxzHTtEBHWKFkr8A?size=1280x960 %}

Currently there are not tests to run. However, Karma will keep an eye out for any files we create in test/** that end in *_spec.js* or *_spec.js.coffee*.

### Testing the Controller

Since we already have the controller *ScreencastsCtrl* created and working, let's write a test around it. Currently the functionality is not what we want in the controller, but it's a good place to start. Also, by starting the testing now, we will be performing **Test Driven Development (TDD)** *#buzzword*.

Create the folder *javascripts* in the test directory.

    $ mkdir -p test/javascripts

Now create a new coffeescript file called *screencasts_ctrl_spec.js.coffee* and add the following:

``` coffeescript test/javascripts/screencasts_ctrl_spec.js.coffee
console.log 'ran @', new Date()

# load the module before each spec
beforeEach module('AngularCasts')

describe "ScreencastsCtrl", ->
  { scope, ctrl } = [ null, null ]

  # get controller and its scope (inherits from `$rootScope`)
  beforeEach inject ($rootScope, $controller)->
    scope = $rootScope.$new()
    ctrl  = $controller "ScreencastsCtrl", $scope: scope

  it 'should set the message', ->
    expect(scope.message).toBe "Angular Rocks!"
```

Once you save the file, your testing browser will reload with the latest test results. If all went well, you should see something like this:

{% img https://photos-6.dropbox.com/t/0/AAADPSUcCWtcIQ0sq_HZoUbB8WA7tLOiofivofkBU_eahA/12/2176587/png/32x32/3/_/1/2/Screen%20Shot%202013-05-02%20at%209.41.09%20PM.png/wpiPrkbqfPbqg32TdbvdYPJ-YV2S_3pl_Bky__FWBpU?size=1280x960 %}

Notice that I have the JavaScript console open. This will allow me to see the logger output *(see line 1 of our controller spec above)*.

#### Confused? Don't worry!

In our test above, the only parts that really counts are lines 14-15. Here is where we are actually testing something. In this case, we are testing to make sure there is a value assigned to the scope variable *message*. It passed because in our controller *(screencasts_ctrl.js.coffee)*, we do assign it a value. Don't believe me? Open up the file and see for yourself.

## TDD Starts Now

In Test Driven Development, the goal is to write a failing test then make the fix after the test has failed. We won't go overboard with this but we will spend a bit of time doing it. Let's get to it!

### Start with the Service

Our controller is going to access the data from our API using [ngResource](http://docs.angularjs.org/api/ngResource.$resource). *ngResource* enables interation with RESTful server-side data sources.

Angular services are singletons that carry out specific tasks common to web apps. Services are commonly used to perform the XHR interaction with the server. Let's start off by creating a service called *screencasts_service.js.coffee*:

``` coffeescript app/assets/javascripts/angular/services/screencast.js.coffee
App.factory 'Screencast', ['$resource', ($resource) ->
  $resource '/api/episodes/:id', id: '@id'
]
```

asdf

``` coffeescript test/javascripts/screencasts_ctrl_spec.js.coffee
console.log 'ran @', new Date()

# load the module before each spec
beforeEach module('AngularCasts')

describe "ScreencastsCtrl", ->
  { scope, ctrl } = [ null, null ]

  # get controller and its scope (inherits from `$rootScope`)
  beforeEach inject ($rootScope, $controller)->
    scope = $rootScope.$new()
    ctrl  = $controller "ScreencastsCtrl", $scope: scope

  it 'should set the message', ->
    expect(scope.message).toBe "Angular Rocks!"
```

## List the Screencasts

Now that we have Angular talking to our views, let's integrate our screencast data. Remember that we can access the JSON data by calling [http://localhost:3000/api/screencasts.json](http://localhost:3000/api/screencasts.json).

Because we are good denizens of happy-coder-land, let's write tests as we go. 


Modify your controller to perform the ajax request and place the results into the `$scope`.

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

``` scss app/assets/stylesheets/episodes.css.scss

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

## Show the Selected Screencast

When we click on the screencast on the left, we want to be able to view the screencast with more information in the main area. This can be done in a few steps:

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

