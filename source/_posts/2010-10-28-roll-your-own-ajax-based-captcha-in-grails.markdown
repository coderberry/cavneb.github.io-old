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

```groovy CaptchaConverter.groovy
package com.berry

import com.berry.BCrypt
import grails.converters.JSON

class CaptchaController {

    def index = {

        // Generate the SALT to be used for encryption and place in session
        def captchaSalt = session.captchaSalt ?: BCrypt.gensalt()
        session.selectedCaptchaText = null
        session.captchaSalt = captchaSalt

        // Modify below for custom images
        def images = [
                'house':        'images/captchaImages/01.png',
                'key':          'images/captchaImages/04.png',
                'flag':         'images/captchaImages/06.png',
                'clock':        'images/captchaImages/15.png',
                'bug':          'images/captchaImages/16.png',
                'pen':          'images/captchaImages/19.png',
                'light bulb':   'images/captchaImages/21.png',
                'musical note': 'images/captchaImages/40.png',
                'heart':        'images/captchaImages/43.png',
                'world':        'images/captchaImages/99.png'
        ]

        // Create the image array to be returned in JSON format
        def size = images.size()
        def num = Math.min(params.numImages ? params.int('numImages') : 5, size)
        def keys = images.keySet().toList()
        def used = []
        def random = new Random()
        (1..num).each { i ->
            def idx = random.nextInt(keys.size())
            def item = keys.get(idx)
            keys.remove(idx)
            used << item
        }

        // Select the 'chosen' text to be used for authentication and place in session
        def selectedText = used[random.nextInt(used.size())]
        def hashedSelectedText = BCrypt.hashpw(selectedText, captchaSalt);
        session.selectedCaptchaText = hashedSelectedText

//        println "SELECTED: ${hashedSelectedText}"
//        println "USED: ${used.inspect()}"

        // Generate object to be returned
        def ret = [
                text: selectedText,
                images: []
        ]
        used.each { u ->
            ret['images'] << [hash: BCrypt.hashpw(u, captchaSalt), file: images[u]]
        }

        render ret as JSON
    }
}
```
    
What this controller does is return a JSON object with the data needed to generate the captcha. The JSON appears like so:

```json data.json
{
  "text":"heart",
  "images":[
    {
      "hash":"$2a$10$GTcG7U1rt7XFBi4JVImT2Oo.E3D8FCzha2772XuXm7v28Kx2LNL5S",
      "file":"images/captchaImages/99.png"
    },
    {
      "hash":"$2a$10$GTcG7U1rt7XFBi4JVImT2Oa5Y/I/cXOUj30kffPqyX0qxTnAACX6O",
      "file":"images/captchaImages/43.png"
    },
    {
      "hash":"$2a$10$GTcG7U1rt7XFBi4JVImT2O8zeOa4.ed1s8pZS9AgkalcSSQm9pmbi",
      "file":"images/captchaImages/15.png"
    },
    {
      "hash":"$2a$10$GTcG7U1rt7XFBi4JVImT2OSNYwC4RPwhNpuPYBbeNB0j4ozoItwDK",
      "file":"images/captchaImages/06.png"
    },
    {
      "hash":"$2a$10$GTcG7U1rt7XFBi4JVImT2OLv6DzHHDX0aB2AwS1YEVZMp9cEpo2sq",
      "file":"images/captchaImages/01.png"
    }
  ]
}
```

Now we just need to implement this in our GSP file and controller. Suppose we have a page like shown above with a pickup code and the last 4 digits of the persons phone number. With adding our captcha div and required javascript, our GSP would look like this:

{% raw %}
```html view.gsp
<!-- PLACE IN HEADER -->
<script type="text/javascript" src="${resource(dir:'js',file:'jquery.simpleCaptcha-0.2.js')}"></script>
<style type="text/css">
    img.simpleCaptcha {
        margin: 2px !important;
        cursor: pointer;
    }
    img.simpleCaptchaSelected {
        margin-bottom: 0px;
        border-bottom: 2px solid red;
    }
</style>
 
<!-- BODY CONTENTS -->
<g:form action="pickup">
 
    <div class="stylized myform" style="width:542px;">
        <h2>Your pickup code will be given to you by your loan consultant</h2>
 
        <g:if test="${flash.message}">
            <div class="error">
                ${flash.message}
            </div>
        </g:if>
 
        <div class="clearfix formField">
            <label class="label_only">Pickup Code</label>
            <g:textField name="pickupCode" value="${pickupCode}" autocomplete="no" class="text" />
        </div>
        <div class="clearfix formField">
            <label class="label_only">Last 4 Digits of Phone</label>
            <span class="after_checkbox" style="padding-right: 0px;">(XXX) XXX-</span>
            <g:textField name="lastFourDigits" value="${lastFourDigits}" autocomplete="no" class="text" maxLength="4" />
        </div>
        <div class="clearfix formField">
            <label class="label_only">Are You Human?</label>
            <div style="float: left; margin-left: 10px;">
              <!-- Begin Captcha -->
                <div id="captcha"></div>
              <!-- End Captcha -->
            </div>
        </div>
        <div class="clearfix" style="margin-top: 15px;">
            <label class="label_only">&nbsp;</label>
            <g:submitButton name="submit" value="Show Me My Offer" class="button" />
        </div>
 
    </div>
 
</g:form>
 
<script type="text/javascript">
    $(document).ready(function() {
        $('#captcha').simpleCaptcha({
            numImages: 5,
            introText: '<p>Are you human? Then pick out the <strong class="captchaText"></strong></p>'
        });
    });
</script>
```
{% endraw %}


<script src="https://gist.github.com/cavneb/1291664"></script>

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
