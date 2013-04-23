---
layout: post
title: "Passing Data From View to Layout via pageProperty"
date: 2010-10-08
comments: true
categories: 
- Grails
- Views
---

I am writing a new Grails application which uses the website template Admintasia. Part of the layout gsp file has a section for the header and sub-header. For this to be used, I needed to be able to pass those two strings from the view to the layout.

Here's what my view looks like:

``` html
<html>
<head>
  <meta name="layout" content="admintasia"/>
  <meta name="pageHeader" content="This is a test" />
  <meta name="pageSubHeader" content="blah blah blah" />
</head>
<body>
...
```
    
And here's what my layout looks like:

``` html
<h2><g:pageProperty name="meta.pageHeader" 
          default="MISSING PAGE HEADER" /></h2>
          
<h4><g:pageProperty name="meta.pageSubHeader" 
          default="" /></h4>
```
    
I hope this helps out!