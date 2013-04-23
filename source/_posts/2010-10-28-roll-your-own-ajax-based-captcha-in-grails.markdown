---
layout: post
title: "Roll your own Ajax-Based Captcha in Grails"
date: 2010-10-28
comments: true
categories: 
- Grails
- Ajax
- Security
---

Recently, I was asked to come up with a better solution to our 
captcha needs. We have been using [ReCaptcha](http://www.google.com/recaptcha), which is great 
but difficult to read at times, and has caused frustrated 
customers and lost sales. I found a great solution at 
[http://www.jkdesign.org/captcha](http://www.jkdesign.org/captcha) which displays a number of 
graphics and let's the user choose the right one to prove they 
are human. Here is a screenshot of my implementation:

{% img /images/posts/Using-Criteria-Builder-with-Projections-1.png %}

To make this work within Grails, I had to make several tweaks. The following files are required: 

*   [JQuery 1.2+](http://jqueryui.com/) (I am using version - 1.4.2)
*   [JQuery UI](http://jqueryui.com/) (I am using version - 1.8.2)
*   [jquery.simpleCaptcha-0.2.js](http://gist.github.com/651605)
*   [Captcha Images](/files/captchaImages.zip) placed in images/captchaImages
*   [BCrypt.java](http://gist.github.com/651613) by Damien Miller
*   CaptchaController.groovy (below)

Create a new controller called Captcha. This can really be named anything, but if you do rename it, it will have to be updated in the jquery.simpleCaptcha-0.2.js file or passed in as an option via the javascript.

{% gist 1291660 %}
    
What this controller does is return a JSON object with the data needed to generate the captcha. The JSON appears like so:

{% gist 1291662 %}
    
Now we just need to implement this in our GSP file and controller. Suppose we have a page like shown above with a pickup code and the last 4 digits of the persons phone number. With adding our captcha div and required javascript, our GSP would look like this:

{% gist 1291664 %}
    
Finally, we need to perform the validation on the controller side. The modified authentication action would look like the following:

``` java
def pickup = {

    // Determine if the captcha is picked correctly
    if (params.captchaSelection != session.selectedCaptchaText) {

      // They selected the correct Captcha image. Continue with Authentication

    } else {
        flash.message = "You did not click the correct image below. Please Try Again."   
    }

}
```
    
So there ya go. It's actually pretty easy and customers seem to like choosing an image much more than typing a word that is difficult to read.
