---
layout: post
title: "AngularJS on Rails 4 - Part 1"
date: 2013-04-22 22:28
comments: true
categories: 
- Rails
- Angular
- JavaScript
---
<div style="width: 242px;
      height: 388px;
      margin: 10px 30px 10px 0;
      float: left;
      background: transparent url(http://f.cl.ly/items/1n0f2g2z2w2U0s2h2t40/angular_mug.jpg) -60px -80px no-repeat;">
</div>

Angular seems to be the big craze as of late. Some may agree and some may not, but AngularJS is one of the next big contenders for being the number one choice of developers. At the time of writing this article, AngularJS is the [12th most watched project on GitHub](https://github.com/languages/JavaScript/most_watched).

Here I want to create a useful Rails application using Angular. The goal is to have a single-page application which allows us to select a screencast link on the left and view it on the right. An example of this would be found at [http://ember101.com](http://ember101.com).

###### Originally I had presented this topic at our local [ruby users group](http://utruby.org). My typical workflow is to write a blog post before presenting and have that post be a reference to my presentation. Since then, I have received a lot of feedback on how I could have enhanced the app. These posts (part 1 and 2) been re-written to reflect those changes. *Special thanks goes to [Tad Thorley](http://tadthorley.com/) for providing the [excellent example application](https://github.com/phaedryx/angularcast-example) based off of the original. Also thanks goes out to those who have commented on these posts.*

<div style="clear: both;"></div>

## Creating the Rails Application

I had a hard time deciding when I began this project on whether to use a full Rails application or a very lightweight ruby web stack like [Sinatra](http://sinatrarb.com). I also experimented with a middle-ground solution called [Rails::API](https://github.com/rails-api/rails-api) (see [Railscast](http://railscasts.com/episodes/348-the-rails-api-gem)). In the end, I used standard Rails (version 4.0.0.rc1). This gave me the flexibility I want; and for the scope of this tutorial I didn't want to distract from learning how to use Angular in an Rails application.

Before anything, we need to create a new Rails application called *Angular Casts*

    $ rails new angular_casts
    ...
    $ cd angular_casts

## Creating the Model and Controller

Our application will be pretty simple. We are only going to worry about storing screencast information in our database. Rails by default comes with a very lightweight database server called SQLite. If you aren't familiar with this, you can visit [http://guides.rubyonrails.org/getting_started.html#configuring-a-database](http://guides.rubyonrails.org/getting_started.html#configuring-a-database) to learn more.

Before we create our model, we need to determine what information we want to store. As a user of this application, I think the most useful would be:

- *title*: What is the name of the screencast?
- *summary*: What is the screencast about?
- *duration*: How long is the screencast?
- *link*: How do I get to the original screencast?
- *published*: When was the screencast published?
- *source*: Who is the provider of the screencast?

Let's create a model and controller based on this information. We need to add the *video_url* field as well, which will be used to embed the video into our app.

    $ rails g resource screencast title summary:text duration link published_at:datetime source video_url

By running the *resource* generator, we now have a model and a controller. The controller will provide our REST API. We also can see that the *screencasts* resources have been added to our routes:

``` ruby config/routes.rb
AngularCasts::Application.routes.draw do
  resources :screencasts
  ...
end
```

Run the migration tasks for both development and test environments:

    $ rake db:migrate; rake db:migrate RAILS_ENV=test

## Testing the Model

###### Testing is not the primary topic of these blog posts, so less time and explanation will be given to them. If you want to learn more about testing, I recommend [http://railscasts.com/episodes/275-how-i-test](http://railscasts.com/episodes/275-how-i-test).

For us to have a long-lasting, maintainable application it is imperative that we keep our code tested. This will also help us stay focused on what our application and code is supposed to do.

Start by running the rake task for testing to see make sure the tests run:

    $ rake test

If all worked well, you should see something like **0 tests, 0 assertions, 0 failures, 0 errors, 0 skips**. This means that the test suite ran, but it didn't find any tests to run.

There are a couple of things we will want to test:

* Make sure that all the required data exists for each screencast.
* Make sure that we do not have two of the same screencast (duplicates).

Before we write our tests, let's update our fixtures file with some testable data:

``` yaml test/fixtures/screencasts.yml
# Read about fixtures at http://api.rubyonrails.org/classes/ActiveRecord/Fixtures.html

fast_rails_commands:
  title: "Fast Rails Commands"
  summary: "Rails commands, such as generators, migrations, and tests, have a tendency to be slow because they need to load the Rails app each time. Here I show three tools to make this faster: Zeus, Spring, and Commands."
  duration: "8:06"
  link: "http://railscasts.com/episodes/412-fast-rails-commands"
  published_at: "Thu, 04 Apr 2013 00:00:00 -0700"
  source: "railscasts"
  video_url: "http://media.railscasts.com/assets/episodes/videos/412-fast-rails-commands.mp4"

wizard_forms_with_wicked:
  title: "Wizard Forms with Wicked"
  summary: "Creating a wizard form can be tricky in Rails. Learn how Wicked can help by turning a controller into a series of multiple steps."
  duration: "11:57"
  link: "http://railscasts.com/episodes/346-wizard-forms-with-wicked"
  published_at: "Thu, 03 May 2012 00:00:00 -0700"
  source: "railscasts"
  video_url: "http://media.railscasts.com/assets/episodes/videos/346-wizard-forms-with-wicked.mp4"

sending_html_emails:
  title: "Sending HTML Email"
  summary: "HTML email can be difficult to code because any CSS should be made inline. Here I present a few tools for doing this including the premailer-rails3 and roadie gems."
  duration: "5:42"
  link: "http://railscasts.com/episodes/312-sending-html-email"
  published_at: "Mon, 02 Jan 2012 00:00:00 -0800"
  source: "railscasts"
  video_url: "http://media.railscasts.com/assets/episodes/videos/312-sending-html-email.mp4"
```

Open up the auto-generated file *test/models/screencast_test.rb* and add the following tests. If you are using Rails 3.x, the test file will appear under *test/unit/screencast_test.rb.*

``` ruby test/models/screencast_test.rb
require 'test_helper'

class ScreencastTest < ActiveSupport::TestCase
  setup do
    @screencast_defaults = {
      title:        'Facebook Authentication',
      summary:      'This will show how to create a new facebook application and configure it. Then add some authentication with the omniauth-facebook gem and top it off with a client-side authentication using the JavaScript SDK.',
      duration:     '12:09',
      link:         'http://railscasts.com/episodes/360-facebook-authentication',
      published_at: Date.parse('Mon, 25 Jun 2012 00:00:00 -0700'),
      source:       'railscasts',
      video_url:    'http://media.railscasts.com/assets/episodes/videos/360-facebook-authentication.mp4'
    }
  end

  test "should be invalid if missing required data" do
    screencast = Screencast.new
    assert !screencast.valid?
    [:title, :summary, :duration, :link, :published_at, :source, :video_url].each do |field_name|
      assert screencast.errors.keys.include? field_name
    end
  end

  test "should be valid if required data exists" do
    screencast = Screencast.new(@screencast_defaults)
    assert screencast.valid?
  end

  test "should only allow one screencast with the same video url" do
    screencast = Screencast.new(@screencast_defaults)
    screencast.video_url = screencasts(:fast_rails_commands).video_url
    assert !screencast.valid?
    assert screencast.errors[:video_url].include? "has already been taken"
  end
end
```

Once this is in place, run the tests again with the command `rake test`.

``` bash
$ rake test
Run options: --seed 29768

# Running tests:

F.F

Finished tests in 0.048601s, 61.7271 tests/s, 61.7271 assertions/s.

  1) Failure:
ScreencastTest#test_should_be_invalid_if_missing_required_data [../angular_casts/test/models/screencast_test.rb:18]:
Failed assertion, no message given.

  2) Failure:
ScreencastTest#test_should_only_allow_one_screencast_with_the_same_video_url [../angular_casts/test/models/screencast_test.rb:32]:
Failed assertion, no message given.

3 tests, 3 assertions, 2 failures, 0 errors, 0 skips
```

You can see we have have 3 tests with 3 assertions and 2 failures. The reason we didn't get 3 failures is because the second test, "should be valid if required data exists", will always pass until we set up some restrictions on the model.

Let's update the model to make these tests pass.

``` ruby app/models/screencast.rb
class Screencast < ActiveRecord::Base
  validates_presence_of :title, :summary, :duration, :link, :published_at, :source, :video_url
  validates_uniqueness_of :video_url
end
```

Run the tests again to see them pass successfully!

## Importing the Video Data

Because we are going to import video feeds from external sites, we need to use a feed parsing library. The best one available is [feedzirra](https://github.com/pauldix/feedzirra). Let's add it to our Gemfile and remove *turbolinks*, *jbuilder* and *sdoc*. We also remove *jquery-rails* because we will use it via [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) instead of inside the Asset Pipeline. This will be further explained in part 2.

``` ruby Gemfile
source 'https://rubygems.org'

gem 'rails',        '4.0.0.rc1'
gem 'sqlite3'
gem 'sass-rails',   '~> 4.0.0.rc1'
gem 'uglifier',     '>= 1.3.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'feedzirra'
```

Now install the gems:

    $ bundle install

### Create an Import Library

Now that we have a place to put all of the screencast information, we need to be able to import it from external feeds. Here we are going to create a simple Ruby class that uses the *feedzirra* gem to grab the feed data, parse it and then add it to our database.

Let's start off by creating a new class called *ScreencastImporter*. Paste the following code into *lib/screencast_importer.rb*.

``` ruby lib/screencast_importer.rb
require 'feedzirra'

class ScreencastImporter
  def self.import_railscasts

    # because the Railscasts feed is targeted at itunes, there is additional metadata that
    # is not collected by Feedzirra by default. By using add_common_feed_entry_element,
    # we can let Feedzirra know how to map those values. See more information at
    # http://www.ruby-doc.org/gems/docs/f/feedzirra-0.1.2/Feedzirra/Feed.html
    Feedzirra::Feed.add_common_feed_entry_element(:enclosure, :value => :url, :as => :video_url)
    Feedzirra::Feed.add_common_feed_entry_element('itunes:duration', :as => :duration)

    # Capture the feed and iterate over each entry
    feed = Feedzirra::Feed.fetch_and_parse("http://feeds.feedburner.com/railscasts")
    feed.entries.each do |entry|

      # Strip out the episode number from the title
      title = entry.title.gsub(/^#\d{1,}\s/, '')

      # Find or create the screencast data into our database
      Screencast.where(video_url: entry.video_url).first_or_create(
        title:        title,
        summary:      entry.summary,
        duration:     entry.duration,
        link:         entry.url,
        published_at: entry.published,
        source:       'railscasts' # set this manually
      )
    end

    # Return the number of total screencasts for the source
    Screencast.where(source: 'railscasts').count
  end
end
```

Note that on lines 17-18, we strip out the episode number from the Railscast title. So "#412 Fast Rails Commands" would become "Fast Rails Commands". See [this Rubular](http://rubular.com/r/duWf3x2mSp) to see how I determined the RegExp pattern.

Now if we were to go into our Rails console, we could trigger this import manually. Give it a shot!

    $ rails c
    Loading development environment (Rails 4.0.0.rc1)

    >> require 'screencast_importer'
    => true

    >> ScreencastImporter.import_railscasts
    .... lots ... of ... feedback ....
    => 345

    >> Screencast.count
    => 345

###### At the time of writing this article, there were 345 public Railscasts. This number will increase as time goes on.

### Trigger Import via Rake

Instead of loading up the Rails console and performing the import manually, it would me much easier to have a simple command that we could run to perform the imports. This is where [rake tasks](http://rake.rubyforge.org/) come in. Let's create a rake task that will perform the import.

``` ruby lib/tasks/screencast_sync.rake
require 'screencast_importer'

namespace :screencast_sync do
  desc 'sync all missing screencasts from Railscasts.com'
  task :railscasts => :environment do
    total = ScreencastImporter.import_railscasts
    puts "There are now #{total} screencasts from Railscasts.com"
  end
end
```

Now that we have our rake task set up, go ahead and run the command:

    $ rake screencast_sync:railscasts
    There are now 345 screencasts from Railscasts.com

It worked! But no time to celebrate.. let's move on.

## Making Episodes Accessible via API

Because we are planning on using Angular for our front-end, we only need to expose our data as JSON. This will allow Angular to talk to the backend via ajax calls.

We are going to only use two calls to the API:

- **/screencasts.json** - returns a full list of episodes
- **/screencasts/ID.json** - returns data for a specified screencast (where ID is the unique ID of the screencast in our db)

Because we used the *resource* generator, we already have our controller and routes. 

Let's do some cleanup to the routes and make sure we only are allowing what we want to use. On top of that, let's [scope](http://guides.rubyonrails.org/routing.html#controller-namespaces-and-routing) our calls to the API with *api*.

``` ruby config/routes.rb
AngularCasts::Application.routes.draw do
  scope :api do
    get "/screencasts(.:format)" => "screencasts#index"
    get "/screencasts/:id(.:format)" => "screencasts#show"
  end
end
```

Run the command `rake routes` to see our changes.

    $ rake routes
    Prefix Verb URI Pattern                    Controller#Action
     GET /api/screencasts(.:format)     screencasts#index
     GET /api/screencasts/:id(.:format) screencasts#show

Now update the controller to render the correct JSON data for the two URL's.

``` ruby app/controllers/screencast_controller.rb 
class ScreencastsController < ApplicationController
  # GET /screencasts
  # GET /screencasts.json
  def index
    render json: Screencast.all
  end

  # GET /screencasts/:id
  # GET /screencasts/:id.json
  def show
    render json: Screencast.find(params[:id])
  end
end
```

Now start up your Rails application and visit this link: [http://localhost:3000/api/screencasts.json](http://localhost:3000/api/screencasts.json). If all went well, you should see JSON data. You should also be able to view [http://localhost:3000/api/screencasts/1.json](http://localhost:3000/api/screencasts/1.json) and see the data belonging to a single screencast.

``` javascript http://localhost:3000/api/screencasts/1.json
{
  "id": 1,
  "title": "Upgrading to Rails 4",
  "summary": "With the release of Rails 4.0.0.rc1 it's time to try it out and report any bugs. Here I walk you through the steps to upgrade a Rails 3.2 application to Rails 4.",
  "duration": "12:44",
  "link": "http://railscasts.com/episodes/415-upgrading-to-rails-4",
  "published_at": "2013-05-06T07:00:00.000Z",
  "source": "railscasts",
  "video_url": "http://media.railscasts.com/assets/episodes/videos/415-upgrading-to-rails-4.mp4",
  "created_at": "2013-05-21T18:22:29.719Z",
  "updated_at": "2013-05-21T18:22:29.719Z"
}
```

## Testing the API

Of course we are going to test the API! It actually isn't as complicated as it may seem. I am not going to go over much explanation beyond the inline comments.

Create a new integration test at *test/integration/api_screencasts_test.rb*.

``` ruby test/integration/api_screencasts_test.rb
require 'test_helper'

class ApiScreencastsTest < ActionDispatch::IntegrationTest
  test "get /api/screencasts.json" do
    get "/api/screencasts.json"
    assert_response :success
    assert body == Screencast.all.to_json
    screencasts = JSON.parse(response.body)
    assert screencasts.size == 3 # because there are three fixtures (see screencasts.yml)
    assert screencasts.any? { |s| s["title"] == screencasts(:fast_rails_commands).title }
  end

  test "get /api/screencasts/:id" do
    screencast = screencasts(:fast_rails_commands)
    get "/api/screencasts/#{screencast.id}.json"
    assert_response :success
    assert body == screencast.to_json
    assert JSON.parse(response.body)["title"] == screencast.title
  end
end
```

Go ahead and run your tests:

    $ rake test
    ...
    5 tests, 18 assertions, 0 failures, 0 errors, 0 skips

Looks like our test are passing.

---

Let's stop for now. Our next steps will be getting our hands dirty with AngularJS.

#### Go to [Part 2](/blog/2013/04/23/angularjs-on-rails-4-part-2/)