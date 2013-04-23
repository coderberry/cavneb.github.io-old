---
layout: post
title: "Bootstrapping AngularJS with Yeoman"
date: 2013-02-13 16:13
comments: true
categories:
- AngularJS
- JavaScript
---

*Note: This post is incomplete and will be worked on over the next week*

<div style="width: 242px;
      height: 388px;
      margin: 10px 30px 10px 0;
      float: left;
      background: transparent url(/images/posts/angular-yeoman.png) top left no-repeat;">
</div>

**[AngularJS](http://angularjs.org/) is awesome.** 

If you are here, you already know that. If not, [watch these videos](http://www.youtube.com/user/angularjs).

**[Yeoman](http://yeoman.io) is awesome.** 

If you are here, you *might* not know that already. It is similar to [middleman](http://middlemanapp.com) but is Node driven instead of ruby. For a kickstart, watch the [screencast by Addi Osmani](http://www.youtube.com/watch?feature=player_embedded&v=vFacaBinGZ0). 

Yeoman also provides support for easy scaffolding, auto-compilation of CoffeeScript and Compass, live preview servers and more. Yup, I was just listing some of the features from [their site](http://yeoman.io/).

Combine these together and you have an excellent solution to bootstrapping your AngularJS applications.

## Install Yeoman

Install Yeoman with the following command:

    $ curl -L get.yeoman.io | bash

Once this is done, you can confirm the install and version:

    $ yeoman --version
    yeoman  v0.9.1

## AngularJS Generators

The generators that Yeoman provides for AngularJS can be found on [GitHub](https://github.com/yeoman/generator-angular). To view the available generators, type `$ yeoman init --help`. Here are the Angular related generators

#### angular:controller

    Usage:
      yeoman init angular:controller NAME [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false

    Description:
        Creates a new Angular controller

    Example:
        yeoman init angular:controller Thing

        This will create:
            app/scripts/controllers/thing-ctrl.js

#### angular:filter

    Usage:
      yeoman init angular:filter NAME [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false

    Description:
        Creates a new AngularJS filter

    Example:
        yeoman init angular:filter thing

        This will create:
            app/scripts/filters/thing.js

#### angular:route

    Usage:
      yeoman init angular:route NAME [options]

    Options:
      -h, --help                # Print generator's options and usage  
                                # Default: false
          --angular:controller  # Angular:controller to be invoked     
                                # Default: 
          --angular:view        # Angular:view to be invoked           
                                # Default: 

    Description:
        Creates a new AngularJS route

    Example:
        yeoman init angular:route thing

        This will create:
            app/scripts/controllers/thing.js
            app/views/thing.html
        And add routing to:
            app.js

#### angular:service

    Usage:
      yeoman init angular:service NAME [type] [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false

    Description:
        Creates a new AngularJS service.
        Docs: http://docs.angularjs.org/guide/dev_guide.services.creating_services

    Example:
        yeoman init angular:service thing

        This will create:
            app/scripts/services/thing.js

#### angular:view

    Usage:
      yeoman init angular:view NAME [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false

    Description:
        Creates a new AngularJS view

    Example:
        yeoman init angular:view Thing

        This will create:
            app/scripts/views/thing.html

#### angular:directive

    Usage:
      yeoman init angular:directive NAME [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false

    Description:
        Creates a new Angular directive

    Example:
        yeoman init angular:directive thing

        This will create:
            app/scripts/directives/thing.js

#### angular:app

    Usage:
      yeoman init angular:app [options]

    Options:
      -h, --help  # Print generator's options and usage  
                  # Default: false  

    Description:
        Create a base Angular application

    Example:
        yeoman init angular:app

        This will create:
            app/.htaccess
            app/404.html
            app/favicon.ico
            app/robots.txt
            app/scripts/vendor/angular.js
            app/scripts/vendor/angular.min.js
            app/styles/main.css
            app/views/main.html
            Gruntfile.js
            package.json
            test/lib/angular-mocks.js
            app/scripts/thing.js
            app/index.html

#### angular:all

    Usage:
      yeoman init angular:all [options]

    Options:
      -h, --help                # Print generator's options and usage  
                                # Default: false
          --angular:app         # Angular:app to be invoked            
                                # Default: 
          --angular:controller  # Angular:controller to be invoked     
                                # Default: 
          --testacular:app      # Testacular:app to be invoked         
                                # Default: 

    Description:
        Creates a complete starter Angular application with a main 
        controller, view and tests

    Example:
        yeoman init angular

        This will create:
            app/.htaccess
            app/404.html
            app/favicon.ico
            app/robots.txt
            app/scripts/vendor/angular.js
            app/scripts/vendor/angular.min.js
            app/styles/main.css
            app/views/main.html
            Gruntfile.js
            package.json
            test/lib/angular-mocks.js
            app/scripts/thing.js
            app/index.html
            app/scripts/controllers/main.js
            test/spec/controllers/main.js
            testacular.conf.js

## Bootstrap your app

<div style="width: 170px;
      height: 200px;
      margin: 10px 0 10px 30px;
      float: right;
      background: transparent url(/images/posts/instant-gratification.png) top left no-repeat;">
</div>

The first thing that must be done is create a base folder for the application.

    $ mkdir my_app
    $ cd my_app/

Now run the **angular:all** generator:

    $ yeoman init angular:all

Finally, start up your server by running
    
    $ yeoman server

Now when you make code changes, it will update live to the app.

## TBD

* Enhance the documentation listed above.
* Show how to use the yo CLI tool
* Use yeoman to create CoffeeScript instead of JavaScript




