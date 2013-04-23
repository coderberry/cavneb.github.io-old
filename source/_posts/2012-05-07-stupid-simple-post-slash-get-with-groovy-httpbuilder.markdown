---
layout: post
title: "Stupid Simple POST/GET with Groovy HTTPBuilder"
date: 2012-05-07 16:57
comments: true
categories: 
- Groovy
- Grails
---

I was frustrated as hell today finding [examples](http://groovy.329449.n5.nabble.com/HTTP-Builder-Post-Example-td366011.html) [on](http://www.javaworld.com/community/node/3017) [how](http://groovy.codehaus.org/modules/http-builder/doc/post.html) [to](http://codingandmore.blogspot.com/2011/07/groovy-http-builder-fetching-json-data.html) [use](http://www.javaworld.com/community/node/3017) [HTTPBuilder](http://groovy.codehaus.org/modules/http-builder/home.html) to perform a simple POST and GET request in my Grails application.

I now have something that I can use. This code has a dependency on `groovyx.net.http` libraries. This is available without even thinking by including the [Rest Plugin](http://grails.org/plugin/rest) into your app.

{% codeblock grails-app/conf/BuildConfig.groovy lang:java %}
    plugins {
        ...
        runtime ":rest:0.7"
        ...
{% endcodeblock %}

Here's the code. I placed it in `/src/groovy/com/berry/utils/ApiConsumer.groovy`

{% codeblock src/groovy/com/berry/utils/ApiConsumer.groovy lang:java %}
package com.berry.utils

import groovyx.net.http.HTTPBuilder
import groovyx.net.http.ContentType
import groovyx.net.http.Method
import groovyx.net.http.RESTClient

class ApiConsumer {

    static def postText(String baseUrl, String path, query, method = Method.POST) {
        try {
            def ret = null
            def http = new HTTPBuilder(baseUrl)

            // perform a POST request, expecting TEXT response
            http.request(method, ContentType.TEXT) {
                uri.path = path
                uri.query = query
                headers.'User-Agent' = 'Mozilla/5.0 Ubuntu/8.10 Firefox/3.0.4'

                // response handler for a success response code
                response.success = { resp, reader ->
                    println "response status: ${resp.statusLine}"
                    println 'Headers: -----------'
                    resp.headers.each { h ->
                        println " ${h.name} : ${h.value}"
                    }

                    ret = reader.getText()

                    println 'Response data: -----'
                    println ret
                    println '--------------------'
                }
            }
            return ret

        } catch (groovyx.net.http.HttpResponseException ex) {
            ex.printStackTrace()
            return null
        } catch (java.net.ConnectException ex) {
            ex.printStackTrace()
            return null
        }
    }

    static def getText(String baseUrl, String path, query) {
        return postText(baseUrl, path, query, Method.GET)
    }

}
{% endcodeblock %}

With this new class, we can easily perform requests.

```java
def url = "http://myexample.com"
def path = "/path/to/api"
def query = [ firstName: "Eric", lastName: "Berry", email: "cavneb@gmail.com" ]

// Submit a request via GET
def response = ApiConsumer.getText(url, path, query)

// Submit a request via POST
def response = ApiConsumer.postText(url, path, query)
```

I hope this helps.

