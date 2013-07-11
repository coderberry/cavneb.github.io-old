---
layout: post
title: "Authentication with EmberJS - Part 1"
date: 2013-07-08 15:24
comments: true
categories: 
- EmberJS
- Rails
- Rails::API
---

{% img /images/posts/keymaster.jpg %}

Authentication with Ember is difficult. I have spent a couple of weeks trying out different approaches and failing time and again. With the help of [Ryan Florence](http://ember101.com) and [Brad Humphrey](https://github.com/wbhumphrey), I have finally been able to understand how it should work and also have built a [simple application](https://github.com/cavneb/simple-auth) which uses it.

My goal in this article will be to build a simple Ember application with a RESTful backend (in Rails) which provides authentication and user registration. We will also set all requests to pass the access token to our backend for authorization.

Here are a couple of the resources I used to build this app:

* [Ember Tools](https://github.com/rpflorence/ember-tools)
* [http://www.embercasts.com](http://www.embercasts.com/episodes/client-side-authentication-part-1)
* [https://github.com/heartsentwined/ember-auth](https://github.com/heartsentwined/ember-auth)
* [http://log.simplabs.com/post/53016599611/authentication-in-ember-js](http://log.simplabs.com/post/53016599611/authentication-in-ember-js)

## Create a Rails API application

Our application is going to be using the [Rails::API](https://github.com/rails-api/rails-api) (see [Railscast](http://railscasts.com/episodes/348-the-rails-api-gem))gem. By using this gem, we limit our Rails app to include only things necessary for API-driven apps. We will also be using Rails 4.0.

    $ gem install rails-api
    $ rails-api new simple_auth --skip-bundle
    $ cd simple_auth

We are going to use the active_model_serializers gem to format our JSON responses to be Ember-friendly. We will also use *has_secure_password* so let's uncomment the 'bcrypt' gem in our Gemfile:

```ruby Gemfile
source 'https://rubygems.org'

gem 'rails', '4.0.0'
gem 'rails-api'
gem 'sqlite3'
gem 'bcrypt-ruby', '~> 3.0.0'
gem 'active_model_serializers'
```

Now install the gems:

    $ bundle install

## Create and test your models

We are going to have two models in our application: **user** and **api_key**. The user will contain the user information including the encrypted password and the api_key will contain the access token and expiration date. The reason we have separated these two tables is to allow a user to have multiple *sessions* at a time.

Create the resources.

    $ rails g resource user name username:string:uniq email:string:uniq password_digest
    ...
    $ rails g resource api_key user:references access_token:string:uniq scope expired_at:datetime created_at:datetime --timestamps=false

Run your migrations:

    $ rake db:migrate; rake db:migrate RAILS_ENV=test

Because we are using the Active Model Serializers gem, serializers are created automatically for our models. However, we want to limit what they return to only the parts which are useful. Update the serializers as follows:

```ruby app/serializers/user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  attributes :id, :name, :username, :email
end
```

```ruby app/serializers/api_key_serializer.rb
class ApiKeySerializer < ActiveModel::Serializer
  attributes :id, :access_token
  
  has_one :user, embed: :id
end

```

Now let's add a couple of tests for our models. Update the fixtures for users so we have a user to work with:

```yaml test/fixtures/users.yml
joe:
  name: Joe User
  username: joe_user
  email: joe_user@example.com
  password_digest: "$2a$10$wJTPdvpGgzDvkXChrcPyqOQrFFawzGu89B1rZze/lVIcJKWiNeAqS" # 'secret'

jane:
  name: Jane User
  username: jane_user
  email: jane_user@example.com
  password_digest: "$2a$10$wJTPdvpGgzDvkXChrcPyqOQrFFawzGu89B1rZze/lVIcJKWiNeAqS" # 'secret'
```

We also want to add a couple of fixtures for the api keys:

```yaml test/fixtures/api_keys.yml
joe_session:
  user: joe
  access_token: <%= SecureRandom.hex %>
  scope: 'session'
  expired_at: <%= 4.hours.from_now %>

jane_api:
  user: jane
  access_token: <%= SecureRandom.hex %>
  scope: 'api'
  expired_at: <%= 30.days.from_now %>
```

Add a test to ensure the api_key generates an access token when created.

```ruby test/models/api_key_test.rb
require 'test_helper'
require 'minitest/mock'

class ApiKeyTest < ActiveSupport::TestCase
  test "generates access token" do
    joe = users(:joe)
    api_key = ApiKey.create(scope: 'session', user_id: joe.id)
    assert !api_key.new_record?
    assert api_key.access_token =~ /\S{32}/
  end

  test "sets the expired_at properly for 'session' scope" do
    Time.stub :now, Time.at(0) do
      joe = users(:joe)
      api_key = ApiKey.create(scope: 'session', user_id: joe.id)
 
      assert api_key.expired_at == 4.hours.from_now
    end
  end
 
  test "sets the expired_at properly for 'api' scope" do
    Time.stub :now, Time.at(0) do
      joe = users(:joe)
      api_key = ApiKey.create(scope: 'api', user_id: joe.id)
 
      assert api_key.expired_at == 30.days.from_now
    end
  end
end
```

For this to pass, we need to update the api_key model:

```ruby app/models/api_key.rb
class ApiKey < ActiveRecord::Base
  validates :scope, inclusion: { in: %w( session api ) }
  before_create :generate_access_token, :set_expiry_date
  belongs_to :user

  scope :session, -> { where(scope: 'session') }
  scope :api,     -> { where(scope: 'api') }
  scope :active,  -> { where("expired_at >= ?", Time.now) }

  private

  def set_expiry_date
    self.expired_at = if self.scope == 'session'
                        4.hours.from_now
                      else
                        30.days.from_now
                      end
  end
  
  def generate_access_token
    begin
      self.access_token = SecureRandom.hex
    end while self.class.exists?(access_token: access_token)
  end
end
```

Run your tests and they should pass:

    $ rake
    ...
    Finished tests in 0.066920s, 44.8296 tests/s, 59.7729 assertions/s.
    3 tests, 4 assertions, 0 failures, 0 errors, 0 skips

Now let's add a test to our user and the accompanying code to make it work:

```ruby test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase
  test "#session" do
    joe = users(:joe)
    api_key = joe.session_api_key
    assert api_key.access_token =~ /\S{32}/
    assert api_key.user_id == joe.id
  end
end

```

```ruby app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password
  has_many :api_keys

  validates :email, presence: true, uniqueness: true
  validates :username, presence: true, uniqueness: true
  validates :name, presence: true

  def session_api_key
    api_keys.active.session.first_or_create
  end
end
```

Tests still pass?

    $ rake
    ...
    Finished tests in 0.080250s, 49.8442 tests/s, 74.7664 assertions/s.
    4 tests, 6 assertions, 0 failures, 0 errors, 0 skips

## API Endpoints

Now that we have our database set up how we want it, let's make it accessible via an API. Here are the parts we want to be able to accomplish:

* Create a new user
* Authenticate an existing user
* Ensure the user is authorized to perform a request (via token)

Let's start off by adding our authorization layer in our Application controller:

```ruby app/controllers/application_controller
class ApplicationController < ActionController::API
  protected

  # Renders a 401 status code if the current user is not authorized
  def ensure_authenticated_user
    head :unauthorized unless current_user
  end

  # Returns the active user associated with the access token if available
  def current_user
    api_key = ApiKey.active.where(access_token: token).first
    if api_key
      return api_key.user
    else
      return nil
    end
  end

  # Parses the access token from the header
  def token
    bearer = request.headers["HTTP_AUTHORIZATION"]

    # allows our tests to pass
    bearer ||= request.headers["rack.session"].try(:[], 'Authorization')
    
    if bearer.present?
      bearer.split.last
    else
      nil
    end
  end
end
```

Now let's set up our users controller:

```ruby app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_filter :ensure_authenticated_user, only: [:index]

  # Returns list of users. This requires authorization
  def index
    render json: User.all
  end

  def show
    render json: User.find(params[:id])
  end

  def create
    user = User.create(user_params)
    if user.new_record?
      render json: { errors: user.errors.messages }, status: 422
    else
      render json: user.session_api_key, status: 201
    end
  end

  private

  # Strong Parameters (Rails 4)
  def user_params
    params.require(:user).permit(:name, :username, :email, :password, :password_confirmation)
  end
end
```

Now create a *session* controller and place our code for authenticating an existing user into it.

    $ rails g controller session

```ruby app/controllers/session_controller.rb
class SessionController < ApplicationController
  def create
    user = User.where("username = ? OR email = ?", params[:username_or_email], params[:username_or_email]).first
    if user && user.authenticate(params[:password])
      render json: user.session_api_key, status: 201
    else
      render json: {}, status: 401
    end
  end
end
```

<div style="background-color: #FDF6E3; padding: 10px; margin-bottom: 10px;">Because RailsAPI application controller extends ActionController::API, it doesn't know about ActionController::StrongParameters. Because of this we need to add an initializer:</div>

```ruby config/initializers/strong_param_fix_for_rails_api.rb
# The application controllers don't know anything about ActionController::StrongParameters 
# because they're not extending the class ActionController::StrongParameters was included within. 
# This is why the require() method call is not calling the implementation 
# in ActionController::StrongParameters
#
# see http://stackoverflow.com/questions/13745689/getting-rails-api-and-strong-parameters-to-work-together

ActionController::API.send :include, ActionController::StrongParameters
```

## Routes

Update your routes file to make sure that it reflects our changes:

```ruby
SimpleAuth::Application.routes.draw do
  resources :users, except: [:new, :edit, :destroy]
  post 'session' => 'session#create'
end
```

## Testing the API

Let's write some tests to make sure our API is functioning as we expect it to. First, let's test out our session controller (for authentication):

```ruby test/controllers/session_controller_test.rb
require 'test_helper'

class SessionControllerTest < ActionController::TestCase
  test "authenticate with username" do
    pw = 'secret'
    larry = User.create!(username: 'larry', email: 'larry@example.com', name: 'Larry Moulders', password: pw, password_confirmation: pw)
    post 'create', { username_or_email: larry.username, password: pw }
    results = JSON.parse(response.body)
    assert results['api_key']['access_token'] =~ /\S{32}/
    assert results['api_key']['user_id'] == larry.id
  end

  test "authenticate with email" do
    pw = 'secret'
    larry = User.create!(username: 'larry', email: 'larry@example.com', name: 'Larry Moulders', password: pw, password_confirmation: pw)
    post 'create', { username_or_email: larry.email, password: pw }
    results = JSON.parse(response.body)
    assert results['api_key']['access_token'] =~ /\S{32}/
    assert results['api_key']['user_id'] == larry.id
  end

  test "authenticate with invalid info" do
    pw = 'secret'
    larry = User.create!(username: 'larry', email: 'larry@example.com', name: 'Larry Moulders', password: pw, password_confirmation: pw)
    post 'create', { username_or_email: larry.email, password: 'huh' }
    assert response.status == 401
  end
end
```

Now, let's add some tests to our users controller (for registration):

```ruby test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase
  test "#create" do
    post 'create', {
      user: { 
        username: 'billy', 
        name: 'Billy Blowers', 
        email: 'billy_blowers@example.com', 
        password: 'secret', 
        password_confirmation: 'secret' 
      }
    }
    results = JSON.parse(response.body)
    assert results['api_key']['access_token'] =~ /\S{32}/
    assert results['api_key']['user_id'] > 0
  end

  test "#create with invalid data" do
    post 'create', {
      user: {
        username: '',
        name: '',
        email: 'foo',
        password: 'secret',
        password_confirmation: 'something_else'
      }
    }
    results = JSON.parse(response.body)
    assert results['errors'].size == 3
  end

  test "#show" do
    joe = users(:joe)
    post 'show', { id: joe.id }
    results = JSON.parse(response.body)
    assert results['user']['id'] == joe.id
    assert results['user']['name'] == joe.name
  end

  test "#index without token in header" do
    get 'index'
    assert response.status == 401
  end

  test "#index with invalid token" do
    get 'index', {}, { 'Authorization' => "Bearer 12345" }
    assert response.status == 401
  end

  test "#index with expired token" do
    joe = users(:joe)
    expired_api_key = joe.api_keys.session.create
    expired_api_key.update_attribute(:expired_at, 30.days.ago)
    assert !ApiKey.active.map(&:id).include?(expired_api_key.id)
    get 'index', {}, { 'Authorization' => "Bearer #{expired_api_key.access_token}" }
    assert response.status == 401
  end

  test "#index with valid token" do
    joe = users(:joe)
    api_key = joe.session_api_key
    get 'index', {}, { 'Authorization' => "Bearer #{api_key.access_token}" }
    results = JSON.parse(response.body)
    assert results['users'].size == 2
  end
end
```

That was a lot! Let's run our tests and make sure everything passes.

    $ rake
    ...........
    Finished tests in 0.229066s, 61.1178 tests/s, 91.6766 assertions/s.
    14 tests, 21 assertions, 0 failures, 0 errors, 0 skips

<h3><a href="/blog/2013/07/08/authentication-with-emberjs-part-2/">Continue to Part 2</a></h3>
