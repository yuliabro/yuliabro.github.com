---
layout: post
category : Selenium
tags : [selenium, webdriver, javascript]
---
{% include JB/setup %}

Executing JavaScript from the test

## Overview

There are several ocations on which executing JavaScript is super helpful when writing Webdriver powered tests.

## 1. Locating page elements

In the ideal world, your page elements will be located with one of the finders determined by Webdriver API:  

    FINDERS =
        {
          :class             => 'class name',
          :class_name        => 'class name',
          :css               => 'css selector',
          :id                => 'id',
          :link              => 'link text',
          :link_text         => 'link text',
          :name              => 'name',
          :partial_link_text => 'partial link text',
          :tag_name          => 'tag name',
          :xpath             => 'xpath',
        }

In most of the cases, :css and :xpath do the job. However, it's not always the case. Imagine a dropdown menu, which only becomes visible on mouse hover.

    <div class="dropdown">
      <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
      </ul>
    </div>

    CSS:
       .dropdown {display: none; position: absolute; z-index: 2}

Even menu items are present in HTML, webdriver will fail to find them. The trick is, the driver only cares for the elements actually visible on the page simply because it is trying to emulate user interactions with the page. It only sees what user can see. 
Driver's ability to execute raw JavaScript can do a job here. Let's click on Item 2 from the list:

     @driver.execute_script("$('.dropdown ul li:nth-child(2) a').click()")

Now, from webdriver's perspective, it might not be "the right way" to do things since indeed, element is not visible and can't be interacted with, but sometimes it's the only way to get an access to elements. Native Mouse events is a better way, but they are not fully supported by ruby webdriver API.

## 2. Making elements visible for further interactions

Make a button visible and continue to interact with it from the driver:

     element :remove_button, :css => "button.remove"
      ....
     @driver.execute_script("$('.remove').css('visibility', 'visible')")
     remove_button.click

## 3. Getting data from the page.

JavaScript execution can return values to the program! Some examples:

     table_cells = @driver.execute_script("return $('table.group_table tbody tr td').size()")

     checkbox_state = @driver.execute_script("return $('input[id=user]:checked').size()")

Will return number of table cells and checkbox state ('1' if checked) respectively.
