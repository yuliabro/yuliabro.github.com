---
layout: post
category : Selenium
tags : [selenium, webdriver]
---
{% include JB/setup %}

Setting Webdriver istance 

## Overview

There are several places in the code where you can set up webdriver instance. The first would be inside of the test:

    class AwesomeTest < Test::Unit::TestCase

      def test_human_can_search_in_google
          @driver = Selenium::WebDriver.for :firefox, :profile => "Selenium"
          ...
      end
    end

Not so practical since you will need an instance of a driver for each test.

The second is setting up a shared instance for the concrete test suite:


    class AwesomeTest < Test::Unit::TestCase

      def setup
          @driver = Selenium::WebDriver.for :firefox, :profile => "Selenium"
      end

      def test_human_can_search_in_google
          ... 
      end
    end

The setup method will be executed before each test which is already better solution.

However, the most optimal way would be to write a module where setup, teardown and bunch of other useful methods could be defined and shared between the tests.  

### Why to bother?

DRY ("Don't Repeat Yourself") is a backbone of writing great, maintainable code. Why testing code should be different from the application code? 

### Webdriver Test Helper

Create a new file, call it webdriver_test_helper.rb and place it into the folder with your selenium tests.

Let's see what we can put inside of our shiny new helper:


      def setup
        # Read env variables
        platform = ENV['PLATFORM']
        url = ENV['SELENIUM_TEST_URL']

        # Apply ultimate logic
        if platform == 'headless'
          headless = Headless.new
          headless.start
          at_exit do
            headless.destroy
          end
        end

        # Define various user types for testing
        @trial_user = 'TrialUser'
        @new_user = 'NewUser'
        @password = 'pass'

        # Setup shared webdriver instance
        @driver = Selenium::WebDriver.for :firefox, :profile => "Selenium"
      end

Run the test from the command line:

     -> SELENIUM_TEST_URL='http://google.com' PLATFORM='headless' ruby test/selenium/awesome_tests.rb 

Setup method is a great place to:
- Read command-line variables and apply the logic accordingly. In the example above, the test suite will run the headless mode (more arbout headless mode later)
- Set up shared @driver instance with desired properties
- Declare variables shared between all tests.  
- If you use remote driver, local vs remote setup logic can be declared there as well (more on remote driver setup later)

Swell, now let's take a peek at the teardown method.

      def teardown
        if !passed?
          take_screenshot(self.to_s)
        end
        logout
        @driver.quit
      end

Here we can see three tiny but useful methods:
- If the test is not passed, take a screenshot of the failed screen. Webdriver lets you do so, why not to use the opportunity?
- Custom method to logout from the user session
- Quit the driver. A nice way to clean up after yourself

### Conclusion

In the end, just include a helper module in your test suite and you are ready to go! 

    # -*- coding: utf-8 -*-
    require "rubygems"
    require "selenium-webdriver"
    require_relative 'selenium_test_helper'

    class AwesomeTest < Test::Unit::TestCase
      include SeleniumTestHelper

      def test_human_can_search_in_google
          ...
      end
    end

