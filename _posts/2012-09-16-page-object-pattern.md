---
layout: post
category : Selenium
tags : [selenium, webdriver]
---
{% include JB/setup %}

This Jekyll introduction will outline specifically  what Jekyll is and why SELENIUM.
Directly following the intro we'll learn exactly _how_ Jekyll does what it does.

## Overview 

### What is Page Object Pattern?

The idea behind the pattern is simple each HTML page of your web-application is represented by a page class that contains methods to interract with the page. The page object is initiated on the test run making page methods available to the test. Each page class is a subclass of the main Page class that contains methods common to all pages. 

### Example
Take this page you are on right now. It consists of the menu, page header, right-side area with listed tags, and a main area with the post body. The actions available from this page are clicking on page links, clicking on tags, leaving a comment to a post. All these actions could be written as page methods.


    require_relative 'page'

    class PostPage < Page

      def click_selenium_tag
        selenium_tag = @driver.find_element(:css => "a[href='/tags.html#selenium-ref']")
        selenium_tag.click
      end

    end


### Why would you want to use Page Object Pattern?

It is a great way to organize your test code. Every page of the app has its own representation, so you know exactly where responsible code lives. You can abstract pages further by creating parent class and make the pages inherit from it. Parent class may contain methods common to all pages, such as interations with menus and footers.

Notice, how so far I escaped mentioning page elements - Page Object Pattern can be used as an abstraction for HTML pages and that is fine. However, if you want a doble gain - the combination of Page Object pattern and Page Elements concept is your shiny double.

### Page Object Patten + Page Elements

Coming back to the example code. Instead of repetitively defining 'selenium_tag' in further page methods, whouldn't it be great to do it only once for the use of the whole page? 


    require_relative 'page'

    class PostPage < Page
    
    element :selenium_tag, :css => "a[href='/tags.html#selenium-ref']"

      def click_selenium_tag
        selenium_tag.click
      end

    end

The advantages are numerous:

- No repetition in defining page elements
- In case element locator changes, you only need to change it in one place in your code
- Elements are listed in one place which is great when you need to come back to add new ones

### Possible implementation
**Using Ruby**

Backbone of the simpliest Page Object + Page Elements framework are two classes: Page and Element.


    class Page

      # method that passes the selenium object when new page is created
      def initialize(driver, test_case)
        @driver = driver
        @test = test_case
        setup_instance_elements(driver)
      end

      # instantiate page elements
      def setup_instance_elements(driver)
        @elements = Hash.new
        self.class.page_elements.each do |name, locator|
          @elements[name.to_sym] = Element.new(name, locator, driver)
        end
      end

      # write elements listed on the page in array
      def self.element(name, locator)
        self.page_elements << [name, locator]

        define_method(name.to_sym) do
          @elements[name.to_sym]
        end
      end

      def self.page_elements
        @page_elements ||= []
      end

      def method_missing(method, *args)
        if /^assert/ =~ method.to_s
          @test.send(method, *args)
        else
          super
        end
      end

    end

The above code accomplishes the following:
- reads the page elements listed on the page and writes them in the array @page_elements
- when a new instance of the page is called from the test, loops through @page_elements array instantiating page element for each element of the array 
- method_missing method checks for the caller's name is "assert" and if so, passes the method and the arguments to the test executor

Page Element class

      class Element

      # method that passes the selenium object
      def initialize(name, locator, driver) 
        @name = name
        @locator = locator
        @driver = driver
      end
      
      # executes webdriver's find_element. If the element is not
      # found, we'll get a nice debugging message

      def element
        @driver.find_element(@locator)
      rescue Selenium::WebDriver::Error::NoSuchElementError
        puts "The following element '#{@name}' with locator '#{@locator}' was not found on the page."
        nil
      end
      
      # displayed?, click, text are methods taken from 
      # Webdriver API. Redefining them here, making applicable 
      # to element instance

      def displayed?
        (e = element) && e.displayed?
      end

      def click
        element.click
      end

      def text
        @driver.find_element(@locator).text
      end

      ...
      # custom methods are combinations of native webdriver commands

      def wait_for_element_and_click(timeout)
        wait = Selenium::WebDriver::Wait.new(:timeout => timeout)
        wait.until { @driver.find_element(@locator).displayed? }
        @driver.find_element(@locator).click
      end
     end

The main purpose of the Element class is proving pages with elements, on which native webdriver API calls can be applied.

Finally, an example of how everything works together by simple test on Google Search functionality. 

Let's start with writing a test, adding pages as we need them. 

    # -*- coding: utf-8 -*-
    require "selenium-webdriver"
    require_relative 'selenium_test_helper'
    require_relative 'page_objects/google_page'

    class WebsiteGoogleTest < Test::Unit::TestCase
      include SeleniumTestHelper #@driver is initiated in the helper

      def test_human_can_search_in_google
          page = GooglePage.new(@driver, self)
          page.open
          page.search_test
      end
    end

The test looks clean and descriptive, as was promissed. Now let's take a look at Google Page class:

    # -*- coding: utf-8 -*-
    require_relative 'page'
    require_relative 'google_results_page'

    class GooglePage < Page
      element :search_input, :css => "input[id='gbqfq']"

      def open
          @driver.get "google.com"
      end

      def search_test
          search_input.send_keys("cheese")
          search_input.send_keys(:enter)
          results_page = GoogleResultsPage.new(@driver, self)  
          results_page.assert_search_result
      end
    end

Some moments to notice here:
- For the purporse of this test we only need to define one page element "search_input", we'll add more elements as we write more tests.
- In search_test method we initiate Google Results page. To stay consistent with Page Object Pattern, each new url should have its own page class. An acual assertion will happen on the results page, which is logical if you think about it.

Now Google Results page:

    # -*- coding: utf-8 -*-
    require_relative 'page'

    class GoogleResultsPage < Page
      element :search_result_div, :css => "div[id='ires']"

      def assert_search_result
          search_result_div.wait_for_element(10)
          assert search_result_div.displayed?, "Oopsy, search result box is not found!"
      end
    end

In the next articles I will talk about setting up driver instance, shared between the tests.
