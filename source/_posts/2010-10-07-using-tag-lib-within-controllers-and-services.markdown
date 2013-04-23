---
layout: post
title: "Using Tag Lib within Controllers and Services"
date: 2010-10-07
comments: true
categories: 
- Groovy
- Grails
- TagLib
---

So let's say you want to utilize the standard grails tag lib 
of formatNumber within your domain class, controller or 
service. How would you go about doing it? With the help of a 
blog post by [Lucas Teixeira](http://lucastex.com.br/2010/02/03/como-acessar-uma-taglib-de-dentro-de-um-service/), I was able to get it working without 
an issue. Here is the code:

``` java
class Book {

  def grailsApplication

  String title
  Float cost

  static constraints = {
    title(blank: false)
    cost(blank: false)
  }

  def String toDisplayString() {
    def g = grailsApplication.mainContext.getBean(
      'org.codehaus.groovy.grails.plugins.web.taglib.ApplicationTagLib'
    )
    return "${title} costs ${g.formatNumber(
      [number: cost, type: "currency", currencyCode: "USD"]
    )}"
  }

}
```

It's also simple to utilize your own tag libs as well. Here's a tag lib that I created to help with XML formatting:

``` java
class CommonTagLib {

    static namespace = 'berry'

    def formatXML = { attrs, body ->
        String rawXML = body().toString()
        try {
            def slurpedXML = new XmlSlurper().parseText(rawXML)
            def writer = new StringWriter()
            new XmlNodePrinter(new PrintWriter(writer)).print(slurpedXML)
            def formattedResponse = writer.toString()
            out << formattedResponse
        } catch(Exception e) {
            log.info "Unable to parse XML: ${e.message}"
            out << rawXML
        }
    }

}
```
    
So to call this tag lib in the domain class, controller or service, I just need to do the following:

``` java
def berry = grailsApplication.mainContext.getBean('CommonTagLib')
return berry.formatXML("....")
```
    
Just make sure that you include the 'def grailsApplication' in the beginning.
