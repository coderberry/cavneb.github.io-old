---
layout: post
title: "Using Criteria Builder with Projections"
date: 2010-10-06
comments: true
categories: 
- Groovy
- Grails
- Gorm
---

Something I had to dig around for today was how to perform a sum on a table using Criteria Builder. It seems that it is treated a bit different than a normal query would be. Here is an example of what I tried and failed at:

``` java
def c = Transaction.createCriteria()
def cnt = c.get {
  projections {
    sum("amount")
  }
  and {
    eq("status", k)
    eq("transactionType", transactionType)
    ge("dateTimeProcessed", cal1.getTime())
    le("dateTimeProcessed", cal2.getTime())
  }
}
```

After playing around with it a bit, I found that I can't use the 'and' closure. I revised the code to the following and it worked:

``` java
def c = Transaction.createCriteria()
def cnt = c.get {
  projections {
    sum("amount")
  }
  eq("status", k)
  eq("transactionType", transactionType)
  ge("dateTimeProcessed", cal1.getTime())
  le("dateTimeProcessed", cal2.getTime())
}
```
