---
layout: post
title: "AngularJS on Rails 4 - Part 2"
date: 2013-04-23 07:35
comments: true
categories: 
- Rails
- Angular
- JavaScript
---
Let's pick up where we left off. If you haven't already, make sure you go through [Part 1](/blog/2013/04/22/angularjs-on-rails-4-part-1/) to create your base Rails app with the API setup.

You can either continue using the code you have created on part 1 or you can catch up by checking out the tagged code:

    $ git clone https://github.com/cavneb/angular_casts
    $ cd angular_casts
    $ git checkout step-1
    $ bundle install
    $ rake db:migrate; rake db:migrate RAILS_ENV=test
    $ rake test
    $ rake screencast_sync:railscasts

## Add Angular Libraries

There are a couple of different ways we can add Angular into our application. [Ryan Bates](http://railscasts.com/episodes/405-angularjs) suggests using the [angular-rails gem](https://github.com/hiravgandhi/angularjs-rails). Even though this is an excellent gem which is well maintained, it's good to know how to do this without a gem.

In our app we are going to link our scripts using a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network). We can find the CDN for Angular at [https://ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular.min.js](https://ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular.min.js). We will also be adding Angular Resource via the CDN as well.

Update your layout file:

{% raw %}
``` html app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>Angular Casts</title>
  <%= stylesheet_link_tag "application", media: "all" %>
  <%= csrf_meta_tags %>
</head>
<body>

  <%= yield %>

  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular.min.js"></script>
  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular-resource.min.js"></script>
  <%= javascript_include_tag "application" %>
</body>
</html>
```
{% endraw %}

### Setup the Javascript Folders

We want to keep our code organized by placing our Angular controllers, filters, services, directives, etc. in the app/assets/javascripts folder. Create the following directories:

* app/assets/javascripts/angular/controllers
* app/assets/javascripts/angular/directives
* app/assets/javascripts/angular/services

Here's a shortcut to do this:

    $ mkdir -p app/assets/javascripts/angular/controllers \
               app/assets/javascripts/angular/directives \
               app/assets/javascripts/angular/services

Now let's create the main javascript file which will drive our Angular application.

``` coffeescript app/assets/javascripts/app.js.coffee
window.App = angular.module('AngularCasts', ['ngResource'])
```

In this file we create a new module called *AngularCasts* and assign it to `window.App`. We also add the dependency of `ngResource` which provides simple REST client functionality.

Next, we need to update our JavaScript manifest to include the our Angular scripts. The order of these is important due to the latter ones depending on the ones prior to them.

``` javascript app/assets/javascripts/application.js
//= require app
//= require_tree ./angular
```

This is quite a change from what exists in the manifest already. We will add jQuery later, but via CDN. You'll see why later in this post.

## Add the View

Next, we need to create a controller. This will allow us to set up a route to a view.

    $ rails g controller home index

This set up the *HomeController* and added the action *index*. Before we modify this view, let's update our layout to acts as an Angular app. This is done by adding the directive `ng-app` to our `<html>` tag:

{% raw %}
``` html app/views/layouts/application.html.erb
<!DOCTYPE html>
<html ng-app>
...
```
{% endraw %}

Now let's update our index view with some simple Angular code:

{% raw %}
``` html app/views/home/index.html.erb
<div>
  <label>Name:</label>
  <input type="text" ng-model="yourName" placeholder="Enter a name here">
  <hr>
  <h1>Hello {{yourName}}!</h1>
</div>
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

Start up your server and open up [http://localhost:3000](http://localhost:3000). Type in your name into the text field. If the content changes as you type, it worked! You now have a functional Angular application!

{% img /images/posts/angular_casts_1.gif %}

###### If you are using Rails 3, you will need to delete the file *public/index.html*

## Now the fun begins!

In order for us to tell the page that it should use the *App* module, we need to add the module name to the `ng-app` *directive*. Set the value of the attribute to *AngularCasts*:

``` html app/views/layouts/application.html.erb
<!DOCTYPE html>
<html ng-app="AngularCasts">
<head>
  <title>Angular Casts</title>
  <%= stylesheet_link_tag "application", media: "all" %>
  <%= csrf_meta_tags %>
</head>
<body>
  <header>
    Angular Casts
  </header>

  <%= yield %>

  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular.min.js"></script>
  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular-resource.min.js"></script>
  <%= javascript_include_tag "application" %>
</body>
</html>
```

Now our view knows to use the AngularCasts module.

###### On lines 9-11 we have added the `<header>` content. Make sure you have this in your layout as well.

## Create an Angular Controller

Let's create a controller that will be used to list out the episodes. Create a new coffeescript file at *app/assets/javascripts/angular/controllers/screencasts_ctrl.js.coffee*

```coffeescript app/assets/javascripts/angular/controllers/screencasts_ctrl.js.coffee
App.controller 'ScreencastsCtrl', ['$scope', ($scope) ->
  $scope.message = "Angular Rocks!"
]
```

On line 1, we create a new Angular controller belonging to *App* named *ScreencastsCtrl*. The controller will be referenced in our view as *ScreencastsCtrl*. For more information on Angular controllers, read [http://docs.angularjs.org/guide/dev_guide.mvc.understanding_controller](http://docs.angularjs.org/guide/dev_guide.mvc.understanding_controller).

Let's update our view to display the message.

{% raw %}
```html app/views/home/index.html.erb
<div ng-controller="ScreencastsCtrl">
  <h1>Message: {{message}}</h1>
</div>
```
{% endraw %}

Here we have bound the contents of the *div* to the controller *ScreencastsCtrl*. Refresh the browser and you should see 'Message: Angular Rocks!'.

## Make it Pretty!

Lets add the much needed CSS to our application. Copy the following into *app/assets/stylesheets/home.css.scss*.

``` scss app/assets/stylesheets/home.css.scss
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
  min-height: 700px;
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
  min-height: 700px;
  padding: 5px 25px;

  #player {
    border: 1px solid #000;
    max-width: 800px;
  }
}
```

Refresh the browser. Ooooh!

## Start with the Service

Our Angular controller is going to access the data from our API using [ngResource](http://docs.angularjs.org/api/ngResource.$resource). *ngResource* enables interation with RESTful server-side data sources.

Angular services are singletons that carry out specific tasks common to web apps. Services are commonly used to perform the XHR interaction with the server. To learn about the differences between services and factories, [read this](http://stackoverflow.com/questions/14324451/angular-service-vs-angular-factory). Let's start off by creating a service at *screencast.js.coffee*:

``` coffeescript app/assets/javascripts/angular/services/screencast.js.coffee
App.factory 'Screencast', ['$resource', ($resource) ->
  $resource '/api/screencasts/:id', id: '@id'
]
```

Now tell the controller to use this service:

``` coffeescript app/assets/javascripts/controllers/screencasts_ctrl.js.coffee
App.controller 'ScreencastsCtrl', ['$scope', 'Screencast', ($scope, Screencast) ->
  $scope.screencasts = Screencast.query()
]
```

Update the index view with the following:

{% raw %}
```html app/views/home/index.html.erb
<div ng-controller="ScreencastsCtrl">
  <div id="screencast-list-container">
    <ul>
      <li ng-repeat="screencast in screencasts">
        <h3>{{screencast.title}} <small>({{screencast.duration}})</small></h3>
      </li>
    </ul>
  </div>
</div>
```
{% endraw %}

Now refresh the page. If all worked well, you should see a list of screencasts on the left side. When we reloaded the page, a *GET* request was sent to */api/screencasts*, populating the *screencasts* attribute in our scope. This is the power of Angular ngResource.

## Now What?

We are doing great! Now we have a list of screencasts on the side which are clickable. However, we don't do anything when they are clicked. What we want to do is show the screencast in the main section along with some additional screencast information.

Start off by adding the HTML code which will be used to display the main content. This is done inside the *index.html.erb* file:

{% raw %}
```html app/views/home/index.html.erb
<div ng-controller="ScreencastsCtrl">
  <div id="screencast-list-container">
    <ul>
      <li ng-repeat="screencast in screencasts">
        <h3>{{screencast.title}} <small>({{screencast.duration}})</small></h3>
      </li>
    </ul>
  </div>

  <div id="screencast-view-container" ng-show="selectedScreencast">
    <h2>{{selectedScreencast.title}}</h2>
    <p>{{selectedScreencast.summary}}</p>
    <p>
      Published at {{selectedScreencast.published_at | date: 'mediumDate'}} 
      - <a ng-href="{{selectedScreencast.link}}">{{selectedScreencast.link}}</a>
    </p>
  </div>
</div>
```
{% endraw %}

On lines 10-17, we have added a div which shows the screencast title and summary. On line 10, we use the [ng-show](http://docs.angularjs.org/api/ng.directive:ngShow) directive which only displays the div if *selectedScreencast* exists.

Go ahead and refresh the page. You should not see any changes. Click on a screencast. Still no changes.

### Click and Show

In order for us to show the main content with the screencast information, we need to do a few things. The first thing we need to do is add an [ng-click](http://docs.angularjs.org/api/ng.directive:ngClick) directive to our screencast list:

{% raw %}
```html app/views/home/index.html.erb
...
  <div id="screencast-list-container" >
    <ul>
      <li ng-repeat="screencast in screencasts"
          ng-click="showScreencast(screencast)">
        <h3>{{screencast.title}} <small>({{screencast.duration}})</small></h3>
      </li>
    </ul>
  </div>
...
```
{% endraw %}

Now when the list item is clicked the function *showScreencast* will be triggered with the screencast being passed in to it. Now let's update our controller with this function:

```coffeescript app/assets/javascripts/angular/controllers/screencasts_ctrl.js.coffee
App.controller 'ScreencastsCtrl', ['$scope', 'Screencast', ($scope, Screencast) ->
  # Attributes accessible on the view
  $scope.screencasts        = Screencast.query()
  $scope.selectedScreencast = null

  # Set the selected screencast to the one which was clicked
  $scope.showScreencast = (screencast) ->
    $scope.selectedScreencast = screencast
]
```

Refresh your browser and click on a screencast. As my wife would incorrectly say: **"Waalah!"**

## Show the Screencast

After doing a bit of looking around, I found that [Flow Player](http://flowplayer.org) offered the easiest and cleanest way to show videos. Let's add the dependent scripts and css links to our layout:

``` html app/views/layouts/application.html.erb
<!DOCTYPE html>
<html ng-app="AngularCasts">
<head>
  <title>Angular Casts</title>
  <link href="//releases.flowplayer.org/5.4.0/skin/minimalist.css" rel="stylesheet">
  <%= stylesheet_link_tag "application", media: "all" %>
  <%= csrf_meta_tags %>
</head>
<body>
  <header>
    Angular Casts
  </header>

  <%= yield %>

  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.8/jquery.min.js"></script>
  <script src="//releases.flowplayer.org/5.4.0/flowplayer.min.js"></script>
  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular.min.js"></script>
  <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.6/angular-resource.min.js"></script>
  <%= javascript_include_tag "application" %>
</body>
</html>
```

Remember how we removed jQuery from our Gemfile and javascript manifest? The reason is because we didn't want it concatenated with the other scripts. FlowPlayer depends on jQuery and so jQuery needs to be available prior to Flow Player being loaded. We also can load it via CDN. Good times for all.

### Create the FlowPlayer Directive

FlowPlayer requires triggering a function *flowplayer()* to show the video. We could add this into our controller, but we love to learn. Let's create a directive which listens to the controller and triggers the *flowplayer* function when *showScreencast* is called.

Create the directive at *app/assets/javascripts/angular/directives/flow_player.js.coffee*

``` coffeescript app/assets/javascripts/angular/directives/flow_player.js.coffee
App.directive 'flowPlayer', ->
  (scope, element, attrs) ->

    # Trigger when the selectedScreencast function is called
    # with a screencast
    scope.$watch 'selectedScreencast', (screencast) ->
      if screencast

        # See http://flowplayer.org/docs/
        element.flowplayer
          playlist: [[mp4: screencast.video_url]]
          ratio: 9 / 14
```

Now add the directive into our view:

{% raw %}
```html app/views/home/index.html.erb
...
  <div id="screencast-view-container" ng-show="selectedScreencast">
    <h2>{{selectedScreencast.title}}</h2>
    <p>{{selectedScreencast.summary}}</p>
    <div flow-player="" id="player"></div>
    <p>
      Published at {{selectedScreencast.published_at | date: 'mediumDate'}} 
      - <a ng-href="{{selectedScreencast.link}}">{{selectedScreencast.link}}</a>
    </p>
  </div>
...
```
{% endraw %}

Refresh your browser and go nuts!

## Extra Goodies

One final thing that I would like to see is some sort of indicator which lets us know which video is playing on the screencast list. This can be done via CSS and some simple code.

In our CSS file, we have already added some style for an active screencast. Any *H3* tag on the side with the class of *active* will show as red. Try it out by adding the class to our view:

{% raw %}
```html app/views/home/index.html.erb
...
  <div id="screencast-list-container" >
    <ul>
      <li ng-repeat="screencast in screencasts"
          ng-click="showScreencast(screencast)">
        <h3 class="active">{{screencast.title}} <small>({{screencast.duration}})</small></h3>
      </li>
    </ul>
  </div>
...
```
{% endraw %}

Refresh the page. You should now see that every screencast link on the left is red.

For us to make it show for the active screencast only we have to make a few changes to our view and controller. Update the view to use the [ng-class](http://docs.angularjs.org/api/ng.directive:ngClass) directive:

 {% raw %}
```html app/views/home/index.html.erb
...
  <div id="screencast-list-container" >
    <ul>
      <li ng-repeat="screencast in screencasts"
          ng-click="showScreencast(screencast, $index)">
        <h3 ng-class="{active: $index == selectedRow}">{{screencast.title}} 
          <small>({{screencast.duration}})</small></h3>
      </li>
    </ul>
  </div>
...
```
{% endraw %}

Note that on line 5 we added the 2nd attribute *&index*. This is available via the *ng-repeat* directive and is the index value of the array (integer). We also modified line 6 to use the *ng-class* directive which only shows "active" if the *$index* is equal to *$scope.selectedRow*.

Now update the controller to work with these changes:

``` coffeescript app/assets/javascripts/controllers/screencasts_ctrl.js.coffee
App.controller 'ScreencastsCtrl', ['$scope', 'Screencast', ($scope, Screencast) ->
  # Attributes accessible on the view
  $scope.selectedScreencast = null
  $scope.selectedRow        = null
  
  # Gather the screencasts and set the selected one to the first on success
  $scope.screencasts = Screencast.query ->
    $scope.selectedScreencast = $scope.screencasts[0]
    $scope.selectedRow = 0

  # Set the selected screencast to the one which was clicked
  $scope.showScreencast = (screencast, row) ->
    $scope.selectedScreencast = screencast
    $scope.selectedRow = row
]
```

We have added the new param *row* to the *showScreencast* function and set this to *$scope.selectedRow*. Refresh your browser and see how things have changed.

## Completed Project

View the working app at [http://angular-casts.herokuapp.com/](http://angular-casts.herokuapp.com/).

## Final Thoughts

I know there are no front-end tests for this tutorial. This was intentional. They were too hard. I spent hours upon hours trying to get the tests working with *$httpBackend* and service testing, etc. I tried *karma* and *testem*. Too much time. Too little benefit. If you are better at this stuff than I am please send me an email or pull request or something with the changes you've made to include the tests. I can always add a *Part 3* which is about nothing other than testing the frontend.

Thank you all who have patiently waited for me to finish this post. I learned never to release a post before it has been proofread and run through several times. Next time I'm going to do a screencast.. just like the [old days](http://www.teachmetocode.com).