---
layout: post
title: "Authentication with EmberJS - Part 1"
date: 2013-07-08 15:24
comments: true
categories: 
- EmberJS
---

{% img /images/posts/keymaster.jpg %}

Authentication with Ember is difficult. I have spent a couple of weeks trying out different approaches and failing time and again. With the help of [Ryan Florence](http://ember101.com) and [Brad Humphrey](https://github.com/wbhumphrey), I have finally been able to understand how it should work and also have built a [simple application](https://github.com/cavneb/simple_auth_with_ember) which uses it.

My goal in this article will be to build a simple Ember application with a RESTful backend (in Rails) which provides authentication and user registration. We will also set all requests to pass the access token to our backend for authorization.

Here are a couple of the resources I used to build this app:

* [Ember Tools](https://github.com/rpflorence/ember-tools)
* [http://www.embercasts.com](http://www.embercasts.com/episodes/client-side-authentication-part-1)
* [https://github.com/heartsentwined/ember-auth](https://github.com/heartsentwined/ember-auth)
* [http://log.simplabs.com/post/53016599611/authentication-in-ember-js](http://log.simplabs.com/post/53016599611/authentication-in-ember-js)

## Create a Rails API application

Our application is going to be using the [Rails::API](https://github.com/rails-api/rails-api) (see [Railscast](http://railscasts.com/episodes/348-the-rails-api-gem))gem. By using this gem, we limit our Rails app to include only things necessary for API-driven apps. We will also be using Rails 4.0.

    $ gem install rails-api
    $ rails-api new simple_auth
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

    $ rails g resource user name username email password_digest
    ...
    $ rails g resource api_key user_id:integer:index access_token scope expired_at:datetime created_at:datetime --timestamps=false

Run your migrations:

    $ rake db:migrate; rake db:migrate RAILS_ENV=test

Now let's add a couple of tests for our models. Update the fixtures for users so we have a user to work with:

```yaml test/fixtures/users.yml
joe:
  name: Joe User
  username: joe_user
  email: joe_user@example.com
  password_digest: something

jane:
  name: Jane User
  username: jane_user
  email: jane_user@example.com
  password_digest: something
```

Add a test to ensure the api_key generates an access token when created.

```ruby test/models/api_key_test.rb
require 'test_helper'

class ApiKeyTest < ActiveSupport::TestCase
  test "generates access token" do
    joe = users(:joe)
    api_key = ApiKey.create(scope: 'session', user_id: joe.id)
    assert !api_key.new_record?
    assert api_key.access_token =~ /\S{32}/
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
                        30.days_from_now
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
    Finished tests in 0.069155s, 14.4603 tests/s, 28.9205 assertions/s.
    1 tests, 2 assertions, 0 failures, 0 errors, 0 skips

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
    Finished tests in 0.077889s, 25.6776 tests/s, 51.3551 assertions/s.
    2 tests, 4 assertions, 0 failures, 0 errors, 0 skips

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

### Because RailsAPI application controller extends ActionController::API, it doesn't know about ActionController::StrongParameters. Because of this we need to add an initializer:

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
    ...
    Finished tests in 0.215704s, 41.7238 tests/s, 64.9038 assertions/s.
    9 tests, 14 assertions, 0 failures, 0 errors, 0 skips

<h3><a href="/blog/2013/07/08/authentication-with-emberjs-part-2/">Continue to Part 2</a></h3>
