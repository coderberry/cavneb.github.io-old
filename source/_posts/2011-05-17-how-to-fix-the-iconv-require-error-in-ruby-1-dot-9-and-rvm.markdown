---
layout: post
title: "How to fix the iconv require error in Ruby 1.9 and RVM"
date: 2011-05-17
comments: true
categories:  
- Ruby
---

*This post was copied from [exceptions.wordpress.com](http://exceptionz.wordpress.com/2010/02/03/how-to-fix-the-iconv-require-error-in-ruby-1-9/) with minor updates.*

So you are working with RVM / Rails 3 / Ruby 1.9.2 and you keep on getting the following error:

```
'require': no such file to load – iconv (LoadError)
```
    
**To fix the issue, perform the following steps:**

1.  Install readline using rvm: rvm install readline rvm package install readline
2.  Now install iconv by executing: rvm install iconv rvm package install iconv
3.  If you already have a version of Ruby 1.9 installed, we need to remove it by executing:
    
```
$ rvm remove 1.9.2
```

4.  The final step is to re-install the version of ruby:

```
$ rvm install --trace 1.9.2 -C --with-iconv-dir=$HOME/.rvm/usr
```

**Now to test that it worked:**

1.  Change to the ruby version you installed: *rvm use 1.9.2*
2.  Check you are on the right version of ruby:

```
$ ruby -v
ruby 1.9.2p180 (2011-02-18 revision 30909) [x86_64-darwin10.6.0]
```

3.  Start a new irb session
4.  Now you should be able to require 'iconv' and get a 'true' result

```
ruby-1.9.2-p180 :001 > require 'iconv'
=> true
```
        
Now you should be good to go!

