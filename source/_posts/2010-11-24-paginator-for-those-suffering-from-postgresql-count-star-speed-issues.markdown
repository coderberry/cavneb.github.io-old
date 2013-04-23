---
layout: post
title: "Paginator for those suffering from PostgreSQL count(*) speed issues"
date: 2010-11-24
comments: true
categories: 
- Grails
- Postgres
---

Our company has been using PostgreSQL for  a very long time and has found it a very solid product for our needs. One thing that we've run into is that as our database grows (1mil+ 
records), there is a major slowdown on page loads due to a [PostgreSQL bug](http://sql-info.de/postgresql/postgres-gotchas.html#1_7). 

To resolve this, I had to re-create a new paginator that didn't rely on the count(\*) sql call. I changed it to work similar to how Google performs their queries relying solely on the offset and max to determine which pagination buttons to show.

Here's my final taglib:

``` java
package com.example

import org.springframework.web.servlet.support.RequestContextUtils as RCU

class PagerTagLib {

  /**
   * Creates next/previous links to support pagination for the current controller
   * This is developed to avoid problems with the PostgreSQL count(*) bug.
   * http://sql-info.de/postgresql/postgres-gotchas.html#1_7
   * 
   * <g:pager total="${accountListInstance.size()}" params="${params}" />
   */
  def pager = { attrs ->
      def writer = out
      if (attrs.total == null) {
          throwTagError("Tag [pager] is missing required attribute [total] which is the total showing for the current page")
      }

      def messageSource = grailsAttributes.messageSource
      def locale = RCU.getLocale(request)

    def total = attrs.int('total') ?: 0

      def action = (attrs.action ? attrs.action : (params.action ? params.action : "list"))
      def offset = params.int('offset') ?: 0
      def max = params.int('max')

      if (!offset) offset = (attrs.int('offset') ?: 0)
      if (!max) max = (attrs.int('max') ?: 10)

      def linkParams = [:]
      if (attrs.params) linkParams.putAll(attrs.params)
      linkParams.offset = offset - max
      linkParams.max = max
      if (params.sort) linkParams.sort = params.sort
      if (params.order) linkParams.order = params.order

      def linkTagAttrs = [action:action]
      if (attrs.controller) {
          linkTagAttrs.controller = attrs.controller
      }
      if (attrs.id!=null) {
          linkTagAttrs.id = attrs.id
      }
      linkTagAttrs.params = linkParams

      // determine paging variables
    def isFirstStep = (offset == 0)
    def isLastStep = (total < max)

      // display previous link when not on firststep
      if (!isFirstStep) {
          linkTagAttrs.class = 'prevLink'
          linkParams.offset = offset - max
          writer << link(linkTagAttrs.clone()) {
              (attrs.prev ? attrs.prev : messageSource.getMessage('paginate.prev', null, messageSource.getMessage('default.paginate.prev', null, 'Previous', locale), locale))
          }
      }

      // display next link when not on laststep
      if (!isLastStep) {
          linkTagAttrs.class = 'nextLink'
          linkParams.offset = offset + max
          writer << link(linkTagAttrs.clone()) {
              (attrs.next ? attrs.next : messageSource.getMessage('paginate.next', null, messageSource.getMessage('default.paginate.next', null, 'Next', locale), locale))
          }
      }
  }

}
```
