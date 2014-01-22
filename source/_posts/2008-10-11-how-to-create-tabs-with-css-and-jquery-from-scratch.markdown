---
layout: post
title: "How to create Tabs with CSS and jQuery from scratch"
date: 2008-10-11
comments: true
categories: 
- CSS
- HTML
- UI
---

Another feature that is often added to Web 2.0 sites is tabbed content. I'm not talking about tabbed navigation. Tabbed content is when different chunks of HTML are shown depending on what tab is clicked.

I am using this on two sites:

<table>
    <tr>
        <td><a href="http://www.partnerfusion.com">{% img /images/posts/tab-1.png %}</a></td>
        <td><a href="http://www.reservationcounter.com">{% img /images/posts/tab-2.png %}</a></td>
    </tr>
</table>

**Goal:** Create a tabbed content window using CSS and [jQuery](http://www.jquery.com/)

If you don't know about jQuery yet, let me have the pleasure to introduce you to it. It's a very easy-to-use JavaScript library that offers easy DOM manipulation, effects and a ton more. You can find out more on jQuery at their website: [http://www.jquery.com](http://www.jquery.com).

*Note: you can also use standard JavaScript or [Prototype](http://www.prototypejs.org/) when doing this as well. I just really like jQuery (can you tell?)*

### Step 1. Get jQuery

In this example, we are going to use the jQuery core library found [here](http://jqueryjs.googlecode.com/files/jquery-1.2.6.min.js).

### Step 2. Create your HTML page

I'm going to create a file called `myapp.html`. I'll put it in the same location that I placed my jQuery library at.

```html
<html>
  <head>
    <title>Tabbed Content with CSS and jQuery</title>
    <link href="myapp.css" rel="stylesheet" type="text/css" />
    <script type="text/javascript" src="jquery-1.2.6.min.js"></script>
    <script type="text/javascript" src="myapp.js"></script>
  </head>
  
  <body>
    
    <!-- This is the box that all of the tabs and contents of 
         the tabs will reside -->
    <div id="tabs_container">
      
      <!-- These are the tabs -->
      <ul class="tabs">
        <li class="active">
          <a href="#" rel="#tab_1_contents" class="tab">Tab 1</a>
        </li>
        <li><a href="#" rel="#tab_2_contents" class="tab">Tab 2</a></li>
        <li><a href="#" rel="#tab_3_contents" class="tab">Tab 3</a></li>
      </ul>
      
      <!-- This is used so the contents don't appear to the 
           right of the tabs -->
      <div class="clear"></div>
      
      <!-- This is a div that hold all the tabbed contents -->
      <div class="tab_contents_container">
    
        <!-- Tab 1 Contents -->
        <div id="tab_1_contents" class="tab_contents tab_contents_active">
          I'm Good Enough!
        </div>
    
        <!-- Tab 2 Contents -->
        <div id="tab_2_contents" class="tab_contents">
          I'm Smart Enough!
        </div>
    
        <!-- Tab 3 Contents -->
        <div id="tab_3_contents" class="tab_contents">
          And Doggone It, People Like Me!
        </div>
    
      </div>
    </div>
    
  </body>
</html>
```

Notice that at the top of the HTML page, we are including three files. The first is a stylesheet that we will use to stylize the tabs and contents. The second is the jQuery library that we downloaded. The third is a small JavaScript file that I included to keep things clean. It's always good practice to separate out your scripts from your HTML pages when possible.

### Step 3. Create your CSS file

Let's create the CSS file that's included in the HTML above. We'll call it `myapp.css`

```css
a:active {
  outline: none;
}
a:focus {
  -moz-outline-style: none;
}
#tabs_container {
  width: 400px;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 12px;
}
#tabs_container ul.tabs {
  list-style: none;
  border-bottom: 1px solid #ccc;
  height: 21px;
  margin: 0;
}
#tabs_container ul.tabs li {
  float: left;
}
#tabs_container ul.tabs li a {
  padding: 3px 10px;
  display: block;
  border-left: 1px solid #ccc;
  border-top: 1px solid #ccc;
  border-right: 1px solid #ccc;
  margin-right: 2px;
  text-decoration: none;
  background-color: #efefef;
}
#tabs_container ul.tabs li.active a {
  background-color: #fff;
  padding-top: 4px;
}
div.tab_contents_container {
  border: 1px solid #ccc;
  border-top: none;
  padding: 10px;
}
div.tab_contents {
  display: none;
}
div.tab_contents_active {
  display: block;
}
div.clear {
  clear: both;
}
```

Lines 1-6 remove the dotted outline that Firefox puts around the clicked tabs.

Lines 7-11 set the width and style properties of the whole container.

Lines 12-17 sets the styles for the tab container. Notice that on line 15, we set a height of 21 pixels. This coincides with the font size set on line 10. If you were to increase or decrease the font size, you will also want to do the same for the height.

Lines 18-20 make it so that all the items in the unordered list (ul tag) appear next to each other instead of on top of each other.

Lines 21-30 sets the tab styles. You can use backgrounds instead of just colors here if desired.

Lines 31-34 sets the style for the active tab. Notice that the top padding is 4 pixels instead of 3. By doing this, we are able to make the bottom of the tab appear white making it look like it's part of the contents below.

Lines 35-39 sets up the style for the contents container. Notice that there is a border set for the right, bottom, and left. This is because our top border is already set on line 14.

Lines 40-42 set the non-active tab contents to be invisible.

Lines 43-45 set the active tab to be visible.

Lines 46-48 are used to clear out any floats that appear above it. (see lines 24-26 in the HTML above)

### Step 4. Create your onLoad JavaScript file

As we did with the CSS file, we are going to create a .js file which contains our custom JavaScript code. We'll put it into `myapp.js`.

```javascript
// Load this script when page loads
$(document).ready(function(){
  
 // Set up a listener so that when anything with a class of 'tab' 
 // is clicked, this function is run.
 $('.tab').click(function () {

  // Remove the 'active' class from the active tab.
  $('#tabs_container > .tabs > li.active')
    .removeClass('active');
    
  // Add the 'active' class to the clicked tab.
  $(this).parent().addClass('active');
    
  // Remove the 'tab_contents_active' class from the visible tab contents.
  $('#tabs_container > .tab_contents_container > div.tab_contents_active')
    .removeClass('tab_contents_active');
    
  // Add the 'tab_contents_active' class to the associated tab contents.
  $(this.rel).addClass('tab_contents_active');
    
 });
});
```

Hopefully my notes in the .js file itself explains how it works well enough.

That should do it. Once you're done, your app should look something like this:

{% img /images/posts/tab-3.png %}

You can download my sample code [here](/files/myapp.zip).

### But Wait, There's More!

Sometimes it doesn't make sense to write this stuff from scratch when there are so many other resources out there to use.

Two that I have used in the past are [JavaScript Tabifier](http://www.barelyfitz.com/projects/tabber/index.php) by Patrick Fitzgerald and [Yahoo UI Tabs](http://developer.yahoo.com/yui/tabview/).

I hope you like this post. If you have any suggestions on future how-to topics you would like me to cover, please email me at cavneb@gmail.com.

