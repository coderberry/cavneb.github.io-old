---
layout: post
title: "Using multiple SMTP accounts with Rails &amp; ActionMailer"
date: 2009-03-30
comments: true
categories: 
- Ruby
- Rails
---

Recently I ran into a problem where I needed to be able to send emails via two different SMTP accounts within the same Rails application. I scoured the net trying to find a method to do this, but I couldn't find one. So I pulled out my hack-hat and got started.

If there are any better ways to do this, I would love to hear about it.

I first created a new YAML file in my config folder called `action_mailer.yml`. In this file, I specified three different nodes with the actionmailer settings.

```yaml
development:
  ...

test:
  ...

production:
  website1:
    domain: "gary@superfriends.com"
    user_name: gary
    password: superduper
    address: smtp.gmail.com
    port: 587
    authentication: :plain

  website2:
    domain: mysupercooldomain.com
    user_name: ABCDEF
    password: blahblah
    address: mail.authsmtp.com
    port: 25
    authentication: :plain
```

Afterwards, I created two mailer models that represent each of the different mailers I will use.

```ruby
# app/models/mailer1.rb

class Website1 < ActionMailer::Base
  def load_settings
    options = YAML.load_file("#{RAILS_ROOT}/config/action_mailer.yml")[RAILS_ENV]["website1"]
    @@smtp_settings = {
      :address              => options["address"],
      :port                 => options["port"],
      :domain               => options["domain"],
      :authentication       => options["authentication"],
      :user_name            => options["user_name"],
      :password             => options["password"]
    }
  end

  def welcome_email(recipient, sent_at = Time.now)
    load_settings
    @subject = 'Thank you for visiting website 1'
    @recipients = RAILS_ENV == "production" ? recipient : "cavneb@gmail.com"
    @from = 'gary@superfriends.com'
    @sent_on = sent_at
  end
end
```

```ruby
# app/models/mailer2.rb

class Website2 < ActionMailer::Base
  def load_settings
    options = YAML.load_file("#{RAILS_ROOT}/config/action_mailer.yml")[RAILS_ENV]["website2"]
    @@smtp_settings = {
      :address              => options["address"],
      :port                 => options["port"],
      :domain               => options["domain"],
      :authentication       => options["authentication"],
      :user_name            => options["user_name"],
      :password             => options["password"]
    }
  end

  def welcome_email(recipient, sent_at = Time.now)
    load_settings
    @subject = 'Thank you for visiting website 2'
    @recipients = RAILS_ENV == "production" ? recipient : "cavneb@gmail.com"
    @from = 'info@mysupercooldomain.com'
    @sent_on = sent_at
  end
end
```

So now when I send an email, I can first determine which mailer to use and then send the email.

For example, your controller might have code that looks like this:

```ruby
# Found in code of controller
if session[:template_name] == "website1"
  Website1.deliver_welcome_email("cavneb@gmail.com")
else
  Website2.deliver_welcome_email("cavneb@gmail.com")
end
```

I realize this is probably the hard way, but hey, it's a start. Please post any plugins or alternatives to doing this if you know of any.