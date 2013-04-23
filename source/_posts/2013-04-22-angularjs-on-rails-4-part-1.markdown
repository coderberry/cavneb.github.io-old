---
layout: post
title: "AngularJS on Rails 4 - Part 1"
date: 2013-04-22 22:28
comments: true
categories: 
- Rails
- AngularJS
- JavaScript
---
<div style="width: 242px;
      height: 388px;
      margin: 10px 30px 10px 0;
      float: left;
      background: transparent url(/images/posts/angular_mug.jpg) -60px -80px no-repeat;">
</div>

AngularJS seems to be the big craze as of late. Some may agree and some may not, but AngularJS is one of the next big contenders for being the number one choice of developers.

Here I want to create a useful Rails application using AngularJS. The goal is to have a single-page application which allows us to select a screencast link on the left and view it on the right. An example of this would be found at [http://ember101.com](http://ember101.com).

We will take it a step further though and set up filtering using search boxes and perhaps more. So let's get started!

<div style="clear: both;"></div>

## Creating the Rails Application

Let's start off by creating a new Rails application called *Angular Casts*

    $ rails new angular_casts

Because we are going to import video feeds from external sites, we need to use a feed parsing library. The best one available is [feedzirra](https://github.com/pauldix/feedzirra). Go ahead and add it to the Gemfile and run `bundle install`.

    gem 'feedzirra'

    $ bundle install

## Importing Data

Now that we have our app in place, we want to be able to import feed data into our application. To do so, we will need to store it in our database.

Create a new model using the *resource* generator. This will generate the controller but not the views. Let's call our model **episode**.

    $ rails g resource episode title description pub_date:datetime video_url link guid duration source

The easiest way for us to import the data is with a rake task. This is a good way to go if we don't plan on doing continuous updates to the feed. The rake task will simply call the importer library that we will write.

```ruby
# lib/tasks/episode_sync.rake

require 'railscast_feed'

namespace :episode_sync do
  desc 'sync all missing episodes from Railscasts.com'
  task :railscasts => :environment do
    RailscastFeed.sync
  end
end
```

Now we need to create the importing functionality in a lib file.

```ruby
# lib/railscast_feed.rb

require 'feedzirra'
class RailscastFeed

  def self.sync
    # add additional elements to be parsed from the feed entries
    Feedzirra::Feed.add_common_feed_entry_element(:enclosure, :value => :url, :as => :video_url)
    Feedzirra::Feed.add_common_feed_entry_element('itunes:duration', :as => :duration)

    feed = Feedzirra::Feed.fetch_and_parse("http://feeds.feedburner.com/railscasts")
    feed.entries.each do |entry|
      Episode.create_from(entry, 'railscasts')
    end
  end

end
```

Finally, let's update our model to create the entries, along with validators to ensure we have good data.

```ruby
# app/models/episode.rb

class Episode < ActiveRecord::Base

  validates :guid, presence: true, uniqueness: [ scope: :source ]
  validates :title, :description, :pub_date, :video_url, :link, :source, presence: true

  def self.create_from(entry, source)
    Episode.where(:guid => entry.entry_id, :source => source).first_or_create(
      title:       entry.title,
      description: entry.summary,
      pub_date:    entry.published,
      video_url:   entry.video_url,
      link:        entry.url,
      duration:    entry.duration
    )
  end

end
```

Now that this is all complete, import the episodes using the rake task we created.

    $ rake episode_sync:railscasts

Congrats! But no time to celebrate.. let's move on.

## Making Episodes Accessible via API

Because we are planning on using AngularJS for our front-end, we only need to expose our data as JSON. This will allow AngularJS to talk to the backend via ajax calls.

We are going to only use two calls to the API: 

- **/episodes.json** - returns a full list of episodes
- **/episodes/ID.json** - returns data for a specified episode (where ID is the unique ID of the episode in our db)

Our routes already include the resources mapping for the episodes. However, let's do some cleanup and make sure we only are allowing what we want to use. We will default the format to 'json' because we will not be using anything else.

```ruby
# config/routes.rb

AngularCasts::Application.routes.draw do
  # resources :episodes
  get '/episodes' => 'episodes#index', format: 'json'
  get '/episodes/:id' => 'episodes#show', format: 'json'
end
```

Now update the controller to render the correct JSON data for the two URL's.

```ruby
# app/models/episodes_controller.rb

class EpisodesController < ApplicationController
  respond_to :json

  def index
    respond_with Episode.all
  end

  def show
    respond_with Episode.find(params[:id])
  end
end
```

Cool. If you are feeling brave, start up your Rails application and visit this link: [http://localhost:3000/episodes.json](http://localhost:3000/episodes.json). If all went well, you should see JSON data. You should also be able to view [http://localhost:3000/episodes/1.json](http://localhost:3000/episodes/1.json) and see the data belonging to a single episode.

Example:

```javascript
{
  "episode": {
    "id": 1,
    "title": "#412 Fast Rails Commands",
    "description": "Rails commands, such as generators, migrations, and tests, have a tendency to be slow because they need to load the Rails app each time. Here I show three tools to make this faster: Zeus, Spring, and Commands.",
    "pub_date": "2013-04-04T07:00:00.000Z",
    "video_url": "http://media.railscasts.com/assets/episodes/videos/412-fast-rails-commands.mp4",
    "link": "http://railscasts.com/episodes/412-fast-rails-commands",
    "guid": "fast-rails-commands",
    "duration": "8:06",
    "source": "railscasts",
    "created_at": "2013-04-23T04:46:32.768Z",
    "updated_at": "2013-04-23T04:46:32.768Z"
  }
}
```

Let's stop for now. Our next steps will be getting our hands dirty with AngularJS.

### Go to [Part 2](/blog/2013/04/23/angularjs-on-rails-4-part-2/)