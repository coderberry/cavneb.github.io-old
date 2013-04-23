---
layout: post
title: "Bunny RabbitMQ"
date: 2011-08-09
comments: true
categories: 
- Ruby
- RabbitMQ
---

{% img /images/posts/crazy-rabbit.jpg %}

Recently I was asked to implement [Redis/Resque](https://github.com/defunkt/resque) into an existing project that already had Redis up and running with another Redis server. This ended up being a lot more difficult than I had anticipated due to the [singleton nature](https://github.com/defunkt/resque/blob/master/lib/resque.rb) of the Resque gem. This led us to explore other possible options and we ended up on [RabbitMQ](http://www.rabbitmq.com/).

RabbitMQ has two heavily supported gems that can be used to interface with our Rails app. One of them is the [AMQP gem](http://rubydoc.info/github/ruby-amqp/amqp/master/file/docs/AMQP091ModelExplained.textile). It is very powerful and acts as a daemon for both publisher and consumer. Our needs aren't as big, however, so we decided to use the [bunny gem](https://github.com/celldee/bunny).

Initially, I started off by wanting to create a publisher/consumer scenario. I installed RabbitMQ on a [VirtualBox](http://www.virtualbox.org/) using [Vagrant](http://vagrantup.com/).

```
$ gem install vagrant # See vangrantup.com for install instructions
$ vagrant box add base http://files.vagrantup.com/lucid32.box
$ vagrant init
```
    
Make sure you modify your Vagrantfile to allow port access for the tcp protocol:

``` ruby    
config.vm.forward_port "tcp", 5672, 5672
```
    
Then I started up the server, logged in and installed the RabbitMQ server:

    $ vagrant up
    $ vagrant ssh
    vagrant@lucid32:~$ sudo apt-get install rabbitmq-server
    ...
    
Once this is done, the RabbitMQ server starts automatically. To ensure that it's working you can run the following command. Remember to run it as sudo.

```
vagrant@lucid32:~$ sudo rabbitmqctl list_queues
Listing queues ...
...done.
vagrant@lucid32:~$
```
    
Now that this works, let's play with the gem. Ensure that you have the bunny gem installed.

```
$ gem install bunny
```
    
Once this is installed, start up a console and we will create a queue.

``` ruby
$ irb
> require 'rubygems'
 => true
> require 'bunny'
 => true
> my_client = Bunny.new
 => #<Bunny::Client:0x1095a..., @port=5672>
> my_client.start
 => :connected 
> my_queue = my_client.queue('my_first_queue')
 => #<Bunny::Queue:0x1063v..., @port=5672>>
```
     
Now we have a queue created. Let's go back to our RabbitMQ server and run the list_queues command again.

```
vagrant@lucid32:~$ sudo rabbitmqctl list_queues
Listing queues ...
my_first_queue  0
...done.
vagrant@lucid32:~$
```
    
As you can see, we now have the *my_first_queue* queue created with 0 items in the queue. 

Let's now add a message to the queue.   

``` ruby
> direct_exchange = my_client.exchange('')
> direct_exchange.publish('This is my first message', :key => my_queue.name
 => nil
> my_queue.message_count
 => 1
```

And now when you run the *list_queues* command, you will see it has a **1** after the *my_first_queue*.

```
vagrant@lucid32:~$ sudo rabbitmqctl list_queues
Listing queues ...
my_first_queue  1
...done.
vagrant@lucid32:~$
```
    
Awesome. Now that we know how to add messages into our RabbitMQ server using the bunny gem, let's now create a consumer to read the messages from the queue.

Let's open up another terminal and run irb. This part will be the same as before.

```
$ irb
> require 'rubygems'
 => true
> require 'bunny'
 => true
> my_client = Bunny.new
 => #<Bunny::Client:0x1095a..., @port=5672>
> my_client.start
 => :connected 
> my_queue = my_client.queue('my_first_queue')
 => #<Bunny::Queue:0x1063v..., @port=5672>>
 ...
> my_queue.message_count
 => 1
```
     
To grab the next item in the queue, we would call *pop* on *my_queue*

``` ruby
> msg = my_queue.pop[:payload]
 => "This is a test message"
> my_queue.message_count
 => 0
```
     
If we want it to run as a listener, we can place the *pop* in a loop.

``` ruby
> loop do
>   if my_queue.message_count > 0
>     msg = my_queue.pop[:payload]
>     puts "Found Message: #{msg}"
>   else
>     sleep 5
>   end
> end
```
    
There you have it!