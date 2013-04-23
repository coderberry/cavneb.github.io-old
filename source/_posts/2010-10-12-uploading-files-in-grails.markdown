---
layout: post
title: "Uploading Files in Grails"
date: 2010-10-12
comments: true
categories:  
- Grails
---

Here's the slides from a presentation I did at the 
[UGGUG](http://groups.google.com/group/uggug) in May 2010. I 
found that I needed to use it again today so I posted it.

<iframe src="http://www.slideshare.net/slideshow/embed_code/4000139" width="565" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>


Here is the source code for the uploader service I created:

``` java
package berry

import org.springframework.web.multipart.MultipartFile
import org.codehaus.groovy.grails.web.context.ServletContextHolder

class FileUploadService {

    boolean transactional = true

    def String uploadFile(MultipartFile file, 
      String name, String destinationDirectory) {

        def servletContext = ServletContextHolder.servletContext
        def storagePath = servletContext.getRealPath(destinationDirectory)

        // Create storage path directory if it does not exist
        def storagePathDirectory = new File(storagePath)
        if (!storagePathDirectory.exists()) {
            print "CREATING DIRECTORY ${storagePath}: "
            if (storagePathDirectory.mkdirs()) {
                println "SUCCESS"
            } else {
                println "FAILED"
            }
        }

        // Store file
        if (!file.isEmpty()) {
            file.transferTo(new File("${storagePath}/${name}"))
            println "Saved file: ${storagePath}/${name}"
            return "${storagePath}/${name}"

        } else {
            println "File ${file.inspect()} was empty!"
            return null
        }
    }
}
```
    
A tip that you may enjoy is the way I implemented this uploader into a real world application. I wanted to be able to upload files into a folder within the web-app folder for development, but in production, I wanted to put it in a folder that is served via Tomcat directly. So on our production server, I created a folder at /opt/assets and created a symbolic link in the $TOMCAT_ROOT/webapps/assets to point to it. I modified the service as follows:

``` java
package berry

import org.springframework.web.multipart.MultipartFile
import org.codehaus.groovy.grails.web.context.ServletContextHolder
import grails.util.GrailsUtil

class FileUploadService {

  boolean transactional = true

  def String uploadFile(MultipartFile file, String name) {
    String storagePath = ""
    if (GrailsUtil.environment == "production") {
      storagePath = "/opt/assets"
    } else {
      def servletContext = ServletContextHolder.servletContext
      storagePath = servletContext.getRealPath('assets')
    }

    // Create storage path directory if it does not exist
    def storagePathDirectory = new File(storagePath)
    if (!storagePathDirectory.exists()) {
      print "CREATING DIRECTORY ${storagePath}: "
      if (storagePathDirectory.mkdirs()) {
        println "SUCCESS"
      } else {
        println "FAILED"
      }
    }

    // Store file
    if (!file.isEmpty()) {
      file.transferTo(new File("${storagePath}/${name}"))
      println "Saved file: ${storagePath}/${name}"
      return "${storagePath}/${name}"
    } else {
      println "File ${file.inspect()} was empty!"
      return null
    }
  }
}
```
    
I then create a folder in my web-app folder named 'assets', being sure to add it to my ignore list for the repository. So once I upload the file, it will save to the correct location.

``` html
<img src="${resource(dir:'assets', file: 'myfile.jpg')}" />
```
    
This now will work in both DEVELOPMENT and PRODUCTION enviroments.

[http://github.com/cavneb/FileUploader](http://github.com/cavneb/FileUploader)
