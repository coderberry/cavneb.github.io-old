---
layout: post
title: "Simple fix for pagination with Twitter Bootstrap"
date: 2012-10-09 16:40
comments: true
categories: 
- Rails
- Tips
---

As a Rails guy, I always perform my table pagination using mislav's [will_paginate](https://github.com/mislav/will_paginate) gem. However, when I use it combined with Twitter Bootstrap, I get an undesired result:

{% img /images/posts/pagination-bad.png %}

There is a very simple fix for this which doesn't require using an [additional gem](https://github.com/yrgoldteeth/bootstrap-will_paginate).

Add the following CSS to your application:

{% codeblock assets/stylesheets/pagination_fix.css lang:css %}
    //-------------------------------------------------------------------------------
    //   Pagination fix for will_paginate and bootstrap
    //-------------------------------------------------------------------------------

    .pagination {
      span.disabled {
        color: #aaa !important;
      }
      em.current {
        float: left;
        padding: 0 14px;
        line-height: 38px;
        text-decoration: none;
        background-color: white;
        border: 1px solid #DDD;
        border-left-width: 0;
      }
      .previous_page {
        border-left: 1px solid #ddd;
      }
    }
{% endcodeblock %}

Refresh your page and the pagination issues should be resolved:

{% img /images/posts/pagination-good.png %}

Hope this helps!

#### Update (Dec 4, 2012):

I recently attempted to use the will_paginate gem with the latest version of Bootstrap (v2.2.1) and the pagination no longer
rendered using the `<li>..</li>` elements. To fix this I added the following into an initializer:

{% codeblock config/initializers/will_paginate.rb lang:ruby %}
if defined?(WillPaginate)
  module WillPaginate
    module ActionView
      def will_paginate(collection = nil, options = {})
        options[:renderer] ||= BootstrapLinkRenderer
        super.try :html_safe
      end

      class BootstrapLinkRenderer < LinkRenderer
        protected

        def html_container(html)
          tag :div, tag(:ul, html), container_attributes
        end

        def page_number(page)
          tag :li, link(page, page, :rel => rel_value(page)), :class => ('active' if page == current_page)
        end

        def previous_or_next_page(page, text, classname)
          tag :li, link(text, page || '#'), :class => [classname[0..3], classname, ('disabled' unless page)].join(' ')
        end

        def gap
          tag :li, link(super, '#'), :class => 'disabled'
        end
      end
    end
  end
end
{% endcodeblock %}

This tip was provided by [Søren Houen](https://github.com/houen) and found on the [Railscasts comments](http://railscasts.com/episodes/329-more-on-twitter-bootstrap?view=comments).
