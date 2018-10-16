---
layout: post
title:  "Jump start a Selenium testing project - Part 1"
author: Cosmin Trăistaru
date:   2016-08-23 15:47:07 +0300
categories: selenium python pytest
image:
    location: /assets/images/start-project.jpg
    alt: "Start Selenium Project"
    width: "1600"
    height: "1200"
---

Every now and then I need to start a new testing project. To give myself an advantage
and to save time I created a dummy project using [python], [selenium], [py.test] and
the pytest plugins [pytest-selenium], [pytest-variables] and [pytest-html]. To make
sure that everything works I integrated the project with [Travis CI].

## Tools that I've used:

[selenium] is a browser automation tools that makes the life of web application testers
 much easier, it's lets you interact with a web page.

[py.test] is a test runner that simplifies the way you write and execute tests in python

[pytest-selenium] is a py.test plugin that creates a selenium fixture
which you can later use to interact with web pages, it keeps the code cleaner
to focus on the main issue the testing. The selenium fixture that is created
by this plugin is function scoped, what this means is that the browser is
closed and reopened with a clean slate for every test, no cache, no cookies.
Which is a good and a bad thing, depending on your needs.

[pytest-variables] is a pytest plugin that lets you pass variables at run time
from a json file. This is a good way to keep you urls, credentials, sensitive information
or settings for a test run without the need to change your code

[pytest-html] is a very nice report tool. It takes the output from pytest and generates a
cool html report that is easy to read and understand. For failed selenium tests
it takes a screen shot and adds it to the report. It also displays the URL,
HTML Browser Log, Driver Log and of course the traceback that generated the failed test.


## So what I've done?

Firstly I created a new virtualenv for my project. Where I've installed the
python packages that
I knew I will use: pytest, selenium, pytest-selenium, pytest-variables
I also added them to the requirements.txt file, pinned them at their latest
current version.
You want your packages pinned to a certain version because you don't know what
changes the future versions of this packages will bring that will break your tests.

Then I added a conftest.py because I wanted to add some settings to the selenium
webdriver and add a virtualdisplay(doesn't support OSX) in order to make the tests
more stable when you run them in CI.

After this I created a new python package that will be the home of our project, with
the name of the project. Base on the Page Object model In this package
I created another two python packages named _pages_ - where will keep the future interactions
with the web pages that we will be testing and _tests_ the place where we are going to put our tests.

Then I created a new _basepage.py_ module in the _pages_ package. A base module that will be inherited by
almost all the future modules that will be in this package. This module contains the basic page interactions
that we will need in the future.
Before each interaction we want to make sure that our element is visible and enabled.
Actions that are covered here: click, enter text, select from a dropdown
In our constructor we are setting the webdriver instance, saving out variables(settings), checking
if we want to load the URL and confirming that we are on the correct page.

{% highlight python %}
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.select import Select
from selenium.webdriver.support.ui import WebDriverWait


class BasePage:
    def __init__(self, selenium, variables, open_url=False):
        self.selenium = selenium
        self.variables = variables
        if open_url:
            self.selenium.get(self.variables['url'])
        self.confirm_page_load()

    def confirm_page_load(self):
        pass

    def click(self, selector):
        element = WebDriverWait(self.selenium, self._IMPLICIT_WAIT).until(
            EC.element_to_be_clickable(
                selector
            )
        )
        element.click()

    def enter_text(self, selector, text):
        element = self.is_visible(selector)
        element.click()
        element.clear()
        element.send_keys(text)

    def select_text_from_dropdown(self, selector, value):
        dropdown = WebDriverWait(
                self.selenium, self._IMPLICIT_WAIT
        ).until(EC.visibility_of_element_located(selector))
        Select(dropdown).select_by_value(value)
{% endhighlight %}

To be continued...

See complete project on Github: [here].

 - Also [Guru99] has a nice Selenium IDE tutorial over on their site.

[Guru99]: https://www.guru99.com/install-selenuim-ide.html
[python]: https://www.python.org/
[selenium]: https://pypi.python.org/pypi/selenium
[py.test]: https://pytest.org/
[pytest-selenium]: https://pytest-selenium.readthedocs.io/
[pytest-variables]: https://pypi.python.org/pypi/pytest-variables
[pytest-html]: https://pypi.python.org/pypi/pytest-html
[Travis CI]: https://travis-ci.org/
[here]: https://github.com/CosminTraistaru/selenium_start
