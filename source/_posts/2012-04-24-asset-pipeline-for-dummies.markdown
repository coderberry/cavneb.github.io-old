---
layout: post
title: "Asset Pipeline for Dummies"
date: 2012-04-24 08:41
comments: true
categories: 
- Rails
- Ruby
---

The Rails asset pipeline is very powerful, but often misunderstood. At the [Utah Ruby User Group](http://utruby.org), most of the attendees aren't sure how to use it fully in their Rails app. It's considered as one of the *magic* features that Rails offers. I admit that I was confused as well and took it's magic for granted. Not any longer. 

I reference the word *asset* a lot in this article. An asset is a file that is to be included in your Rails application (JavaScript, CSS, Image, etc).

**In this article, I want to simplify the asset pipeline so it is better understood.**

## Purpose

The asset pipeline has three goals: precompile, concatenate and minify assets into one central path. Or in other words, it takes all of your stylesheets, javascript files, images and any other files you want, joins them together when possible, and places them in the public/assets folder.

## Moving Parts

<img src="/images/posts/sprockets.png" class="fleft" align="top" />
The asset pipeline is powered by two technologies: [Sprockets](https://github.com/sstephenson/sprockets) and [Tilt](https://github.com/rtomayko/tilt), the latter being a dependency of the former (look at your `Gemfile.lock` if you don't believe me). 

**Sprockets** performs the asset packaging which takes the assets from all the specified paths, compiles them together and places them in the target path (public/assets).

**Tilt** is the template engine that Sprockets uses. This allows file types like *scss* and *erb* to be used in the asset pipeline. See the [Tilt Readme](https://github.com/rtomayko/tilt/blob/master/README.md) for a list of supported template engines.

{% img /images/posts/asset_pipeline_flow.png %}

## Asset Paths

Rails applications default to having three possible asset paths.

`app/assets` is for assets that are owned by the application, such as custom images, JavaScript files or stylesheets.

`lib/assets` is for your own libraries’ code that doesn’t really fit into the scope of the application or those libraries which are shared across applications.

`vendor/assets` is for assets that are owned by outside entities, such as code for JavaScript plugins and CSS frameworks.

## The Manifest

The keystone of the asset pipeline is the manifest file. By default, Rails creates one for stylesheets (`app/assets/stylesheets/application.css`) and JavaScript files (`app/assets/javascripts/application.js`). This file uses *directives* to declare dependencies in asset source files.

For directives that take a path argument, you may specify either a logical path or a relative path. Relative paths begin with ./ and reference files relative to the location of the current file.

Here are some *directives* that can be used:

* `require` *[path]* inserts the contents of the asset source file specified by path. If the file is required multiple times, it will appear in the bundle only once.
* `include` *[path]* works like require, but inserts the contents of the specified source file even if it has already been included or required.
`require_directory` *[path]* requires all source files of the same format in the directory specified by path. Files are required in alphabetical order.
* `require_tree` *[path]* works like require_directory, but operates recursively to require all files in all subdirectories of the directory specified by path.
* `require_self` tells Sprockets to insert the body of the current source file before any subsequent require or include directives.
* `depend_on` *[path]* declares a dependency on the given path without including it in the bundle. This is useful when you need to expire an asset's cache in response to a change in another file.
* `stub` *[path]* allows dependency to be excluded from the asset bundle. The path must be a valid asset and may or may not already be part of the bundle. Once stubbed, it is blacklisted and can't be brought back by any other require.

Documentation for this section was largely extracted from the [Sprockets](https://github.com/sstephenson/sprockets) README.

## Usage

Using the asset pipeline is very simple. All it involves is placing assets (js/css/images/other) into the asset path. You can access the files using multiple helper methods within your views:

```ruby
audio_path("horse.wav")   # => /audios/horse.wav
audio_tag("sound")        # => <audio src="/audios/sound" />
font_path("font.ttf")     # => /fonts/font.ttf
image_path("edit.png")    # => "/images/edit.png"
image_tag("icon.png")     # => <img src="/images/icon.png" alt="Icon" />
video_path("hd.avi")      # => /videos/hd.avi
video_tag("trailer.ogg")  # => <video src="/videos/trailer.ogg" />
```

See [ActionView::Helpers::AssetTagHelper](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html) documentation for more information.

## Misconceptions

#### Files must belong in their respective paths. For example, all JavaScript files must be in a `javascripts` folder within an asset path.

The truth is that the paths (*stylesheets*, *javascripts*, *images*) are only there for organization. You can have all the assets in a single folder or in a hundred.

#### [Sass](http://sass-lang.com/) files need to use *erb* extension to allow for asset path inclusions within the files. 

The truth is that `sass-rails` provides `-url` and `-path` helpers for the following asset types: image, font, video, audio, JavaScript and stylesheet.

```ruby
image-url("rails.png")         # becomes url(/assets/rails.png)
image-path("rails.png")        # becomes "/assets/rails.png".
asset-url("rails.png", image)  # becomes url(/assets/rails.png)
asset-path("rails.png", image) # becomes "/assets/rails.png"
```

See the [Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html#coding-links-to-assets) guide (2.2.2) for more information.

## Adding to Gems

A good way to include assets easily in a Rails application is by using gems. To include assets within a gem to be precompiled with the Rails application that is using it, all you need is to place the assets in one of the three standard asset paths: `app/assets`, `lib/assets` and `vendor/assets`. These will be automatically included in by Sprockets when the assets are compiled. See the [Rails documentation](http://guides.rubyonrails.org/asset_pipeline.html#adding-assets-to-your-gems) for more information.

## FAQ

#### Q: Why doesn't the auto-generated scss and coffeescript only get included in their respective controller views?

Because the assets all concatenate into one file, there are no seperate files to be included on a view-by-view basis. There is a way to get around this by using css classes.

Let's say we have a controller named `Users` with an accompanying sass file called `users.css.scss`. Make sure your css is wrapped in a class which includes the name of the controller:

```css
body-users {
  // Custom css goes here
}
```

Next, add a class to the body tag of your layout:

```html
<body class="body-#{controller_name}">
```

Now the css in `users.css.scss` will only be applied to views under the `Users` controller.

* This may not be best practice. Refer to the comments below.*

#### Q: Do I have to use the asset pipeline?

No. In Rails 3.1, the asset pipeline is enabled by default. It can be disabled in `config/application.rb` by putting this line inside the application class definition:

```ruby
config.assets.enabled = false
```

#### Q: What happens if there are duplicate file names in different asset folders?

Let's say you have two asset files with the same name in different paths. For example, let's say we have two files: `app/assets/stylesheets/style.css.scss` and `vendor/assets/stylesheets/style.css.scss`. 

When the assets are compiled, it disregards all the duplicate files after the first one found in the asset path. Let's look at the asset path using the *rails console*:

```bash
>> y Rails.application.config.assets.paths
---
- /Users/eberry/example/app/assets/images
- /Users/eberry/example/app/assets/javascripts
- /Users/eberry/example/app/assets/stylesheets
- /Users/eberry/example/vendor/assets/javascripts
- /Users/eberry/example/vendor/assets/stylesheets
- /Users/eberry/.rvm/gems/ruby-1.9.2-p290/gems/jquery-rails-2.0.2/vendor/assets/javascripts
- /Users/eberry/.rvm/gems/ruby-1.9.2-p290/gems/coffee-rails-3.2.2/lib/assets/javascripts
```

Note that the path `/Users/eberry/example/app/assets/stylesheets` appears before the path `/Users/eberry/example/vendor/assets/stylesheets`.

#### Q: How can I precompile assets that aren't to be used in the pipeline?

Let's say you want to include the folder `other/assets` into the asset pipeline to be precompiled. This is a simple addition in the `application.rb` file (or environment specific config file).

```ruby
module Foo
  class Application < Rails::Application
    ...
    # Add additional path to the assets path for pipeline compilation
    config.assets.paths << "#{Rails.root}/other/assets"
  end
end
```

Now when you run the command `Rails.application.config.assets.paths` in the Rails console, you will see the new asset path.

#### Q: How can I have certain JavaScript files appear at the bottom of the HTML page?

Multiple manifests can be created in the assets folder. For example, I can have a separate manifest called `footer.js` which includes the files `footer_1.js` and `footer_2.js`.

```javascript
//= require footer_1
//= require footer_2
```

I can add this into the HTML by using the same `javascript_include_tag` that is used in the HTML header of the layout.

```html
    <%= javascript_include_tag("footer") %>
  </body>
</html>
```

#### Q: How can I precompile additional assets without having to include them in the manifest?

Let's say we have a file called `search.js` in our `app/assets/javascripts` directory and we don't include it in the manifest. We still would like it to be compiled and placed into the `public/assets` when the assets are compiled.

This is very simple. Just add the following to your `application.rb` file (or environment specific config file):

```ruby
# Precompile additional assets (application.js, 
# application.css, and all non-JS/CSS are already added)
config.assets.precompile += %w( search.js )
```

This configuration option appears by default in `config/environments/production.rb`.

## Summary

As I said before, the asset pipeline has three goals: **precompile*, *concatenate* and *minify* assets.

**Precompilation** let's you use higher-level languages to create actual assets (for example, Sass to CSS).

**Concatenation** is very important in the production environment. It can reduce the number of requests that a browser makes to render a web page, which leads to faster load time.

**Minification** takes out the extra whitespace and comments from your assets. This allows for smaller asset file size, which leads to faster load times.

I strongly suggest learning more about the asset pipeline by going to the [Rails documentation](http://guides.rubyonrails.org/asset_pipeline.html). Ryan Bates also two excellent Railscasts on [Understanding the Asset Pipeline](http://railscasts.com/episodes/279-understanding-the-asset-pipeline) and [Asset Pipeline in Production](http://railscasts.com/episodes/341-asset-pipeline-in-production).

Feel free to hop on the #urug channel on Freenode to chat with me anytime. Also, for a different perspective on asset handling, see the [Resources](http://grails-plugins.github.com/grails-resources/) plugin for [Grails](http://grails.org).
