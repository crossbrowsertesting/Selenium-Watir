**Getting Started with Watir and Selenium**

[Watir](https://watir.com/) is an open-source web application testing framework that is designed to make writing [Selenium](http://www.seleniumhq.org/) tests simple and efficient. Built on Selenium's [Ruby language bindings](https://rubygems.org/gems/selenium-webdriver), Watir is able to drive the browser in the same way humans do. With all of the awesome features Selenium has to offer, the sky's the limit as far as what you can do. Here, we explain how to incorporate the Watir testing framework with CrossBrowserTesting's cloud automation platform. Let's get started.

If you're already familiar with Watir, and you have written tests, making them work with our tool is easy.

Before:

`browser = Watir::Browser.new`

Now:

```ruby
browser = Watir::Browser.new(
    :remote,
	:url => "http://#{username}:#{authkey}@hub.crossbrowsertesting.com:80/wd/hub",
	:desired_capabilities => caps)
```

Username here is the email address associated with your account, and authkey is the authorization key that can be found on the 'Manage Account' section of our site. Caps is the capabilities object that contains our api names for selecting OS/Browser and other options. Your capabilities should look something like this:

```ruby
caps = Selenium::WebDriver::Remote::Capabilities.new

caps["name"] = "Selenium Test Example"
caps["build"] = "1.0"
caps["browserName"] = "Firefox" 	# Pulls latest version by default
caps["platform"] = "Windows 7"		# To specify a version, add caps["version"] = "desired version"
caps["screen_resolution"] = "1024x768"
caps["record_video"] = "true"
caps["record_network"] = "false"					
````

A complete list of capabilities options can be found here:

If you're new to Watir, and you'd like to start from scratch, you've come to the right place. First, we'll need a couple of necessary Selenium libraries. Getting them is easy with Gem.

`gem install selenium-webdriver`
`gem install watir-webdriver`

We'll also be using [Rest](https://github.com/rest-client/rest-client) to make RESTful API calls with CrossBrowserTesting's API.

`gem install rest-client`

Lastly, let's use test-unit to perform functional unit tests on our platform.

`gem install test-unit`



Alright, we should be ready to get started. Let's build a class called CBT_API that will make working with our API easy:

```ruby
class CBT_API
	@@username = 'you%40yourcompany.com'
	@@authkey = '12345'
	@@BaseUrl =   "https://#{@@username}:#{@@authkey}@crossbrowsertesting.com/api/v3"
	def getSnapshot(sessionId)
	    # this returns the the snapshot's "hash" which is used in the
	    # setDescription function
	    response = RestClient.post(@@BaseUrl + "/selenium/#{sessionId}/snapshots",
	        "selenium_test_id=#{sessionId}")
	    snapshotHash = /(?<="hash": ")((\w|\d)*)/.match(response)[0]
	    return snapshotHash
	end

	def setDescription(sessionId, snapshotHash, description)
	    response = RestClient.put(@@BaseUrl + "/selenium/#{sessionId}/snapshots/#{snapshotHash}",
	        "description=#{description}")
	end

	def setScore(sessionId, score)
	    # valid scores are 'pass', 'fail', and 'unset'
	    response = RestClient.put(@@BaseUrl + "/selenium/#{sessionId}",
	        "action=set_score&score=#{score}")
	end
end
```

Now that we have our API object setup, we can set the score of our test to pass/fail, take snapshots whenever we need to, or even set the description so we can easily search for results later. We can now build our unit test. In this test, we'll navigate to an example To-Do app, we'll use some of the functionality of the app, then we'll assert that the changes we made had the desired effect.

```ruby
class CBT_Example < Test::Unit::TestCase
	def test_todos
		begin
			username = 'you%40yourcompany.com'
			authkey = '12345'

			caps = Selenium::WebDriver::Remote::Capabilities.new

			caps["name"] = "Selenium Test Example"
			caps["build"] = "1.0"
			caps["browserName"] = "Firefox" 	# Pulls latest version by default
			caps["platform"] = "Windows 7"		# To specify a version, add caps["version"] = "desired version"
			caps["screen_resolution"] = "1024x768"
			caps["record_video"] = "true"
			caps["record_network"] = "true"

			browser = Watir::Browser.new(
				:remote,
				:url => "http://#{username}:#{authkey}@hub.crossbrowsertesting.com:80/wd/hub",
				:desired_capabilities => caps)

			session_id = browser.driver.session_id

		    score = "pass"
		    cbt_api = CBT_API.new
		    # maximize the window - DESKTOPS ONLY
		    # driver.manage.window.maximize
		    puts "Loading URL"
		    browser.goto ("http://crossbrowsertesting.github.io/todo-app.html")

		    puts "Clicking Checkbox"
		    # driver.find_element(:name, "todo-4").click
		    browser.checkbox(:name, "todo-4").set
		    puts "Clicking Checkbox"
		    # driver.find_element(:name, "todo-5").click
		    browser.checkbox(:name, "todo-5").set

		    # elems = driver.find_elements(:class, "done-true")
		    elems = browser.elements(:class, "done-true")
		    assert_equal(2, elems.length)

		    puts "Entering Text"
		    browser.text_field(:id => 'todotext').set("run your first selenium test")
		    # driver.find_element(:id, "todotext").send_keys("run your first selenium test")
		    browser.element(:id, 'addbutton').click
		    # driver.find_element(:id, "addbutton").click

		    # spanText = driver.find_element(:xpath, "/html/body/div/div/div/ul/li[6]/span").text
		    spanText = browser.element(:xpath, "/html/body/div/div/div/ul/li[6]/span").text
		    assert_equal("run your first selenium test", spanText, "Problem adding to-do")

		    "Archiving old to-dos"
		    # driver.find_element(:link_text, "archive").click
		    browser.element(:link_text, "archive").click
		    # elems = driver.find_elements(:class, "done-false")
		    elems = browser.elements(:class, "done-false")
		    assert_equal(4, elems.length, "Problem archiving to-dos")

		    puts "Taking Snapshot"
		    cbt_api.getSnapshot(session_id)
		    cbt_api.setScore(session_id, "pass")
		rescue Exception => ex
		    puts ("#{ex.class}: #{ex.message}")
		    cbt_api.setScore(session_id, "fail")
		ensure     
		    # driver.quit
		    browser.close
		end
	end
end
```

As you can see from our test, we interact with the app, and then we assert that those changes were made. Rather than having to test this manually against each configuration, we can now easily run our test against any configuration and simply focus on the tests that failed. Thanks to Watir's webdriver writing and extending these tests are easy and efficient. If you have any trouble, feel free to [get in touch with us](<mailto:info@crossbrowsertesting.com>)! We're always happy to help.
