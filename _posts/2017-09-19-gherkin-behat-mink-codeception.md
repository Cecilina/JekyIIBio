---
title: Gherkin Behat Codeception
teaser: Behat / Codeception TEST Environment
category: test
tags: [Test, Workflow]
---

Behavior Driven Development (BDD) for Symfony (php7.0) using Behat / Codeception with Gherkin Language.

Requirements: Composer
----------------------------------------

### 1.1 Behat Mink mink-goutte-driver mink-selenium2-driver

Use `composer` to generate autoload for project.
Add following library to composer.json file.
> #### Here, I am not use the latest version of Behat since I need make Gherkin library compatible with both Behat and Codeception

{% highlight JSON %}
"require-dev": {
  "behat/behat": "^3.3",
  "behat/mink": "^1.7",
  "behat/mink-browserkit-driver": "^1.3",
  "behat/mink-extension": "^2.2",
  "behat/mink-goutte-driver": "^1.2",
  "behat/mink-selenium2-driver": "^1.3",
  "behat/symfony2-extension": "^2.1"
}
{% endhighlight %}

Then run
~~~
composer update
vendor/bin/behat --init #Initialize
~~~

You will notice that a folder called `features` created.

First, create one file called `behat.yml` under root directory as following:
~~~
# behat.yml
default:
  extensions:
    Behat\Symfony2Extension: ~
    Behat\MinkExtension:
      goutte: ~
      selenium2: ~
      base_url: http://127.0.0.1:8000 # the base url of your site
suites:
  default:
    contexts:
      - FeatureContext
      - Behat\MinkExtension\Context\MinkContext
~~~

Then, create one feature called 'about.feature' inside `features` directory.
Write some Feature inside it as following:
~~~
Feature: about page
  In order to see about page contents
  As a user
  I am able to visit about page

  Scenario: Visiting about page
  Given I am on "/about"
  Then I should see "This is about page."

  @javascript
  Scenario: Visiting about page for an existing user
  Given I am on "/about/john"
  When I press detail button
  Then I should see "email"

  Scenario: Visiting about page for an non-existing user
  Given I am on "/about/jim"
  Then I should see "No user named jim found!"
~~~

As notice these Scenario inside feature are almost same, except one with `@javascript` annotation, which will trigger `selenium2` driver for testing. others will use headless `goutte` driver for testing.

In order to let `selenium2` driver running. First need go to [here][selenium2_download] to download it. As selenium2 default using firefox. we need download `geckodriver` [here][geckodriver_download] to support it.

After download both `selenium2` and `geckodriver`. Run
~~~
sudo java -jar -Dwebdriver.gecko.driver=[directory of file geckodriver] [directory of jar file selenium-server-standalone]
~~~
>#### Since there is an issue with geckodriver test for symfony3. using chromedriver
> download chromedriver [here][chromedriver_download]
> run commands as following to start it
{% highlight shell%}
sudo java -jar -Dwebdriver.gecko.driver=[directory of file chromedriver] [directory of jar file selenium-server-standalone]
{% endhighlight%}

run
~~~
vendor/bin/behat
~~~
You should see some output result.

Sometimes, you need write our own custom feature code. You will notice it when `undefined` shown up in the output. You can run the command as following, which will automatically paste the need code to `FeatureContext` Class @see behat.yml configuration file
~~~
vendor/bin/behat --append-snippets 
~~~

We will need complete the function based on the need. [here][mink_doc] is the Mink Doc, which will be helpful.


### 1.2 Codeception

Use `codeception` we don't need wrtie Scenario in Gherkin. We can write test Scenario in `PHP`.

Delete `tests` folder completely if exist. It will regenerate.

Update `composer.json` as following
~~~
composer require codeception/codeception --dev
vendor/bin/codecept bootstrap
~~~

You will notice three test suit generate `acceptance`, `functional`, `unit`.
And each suit has its own test folder 'tests/acceptance', 'test/functional', 'test/unit'

For symfony functional test. Please edit functional.suit.yml as following:
{% highlight yml %}
actor: FunctionalTester # as a tester passing to functional test class function
modules:
    enabled:
        # add a framework module here
        - Symfony:
          app_path: 'app'
          var_path: 'var'
          environment: 'test'
{% endhighlight %}

There are several commands to generate test class as following:
You will notice that the [CLASSNAME] will automatically append test type
~~~
vendor/bin/codecept g:test unit [CLASSNAME] # more function than classic phpunit
vendor/bin/codecept g:cest functional [CLASSNAME]
vendor/bin/codecept g:cept functional [CLASSNAME] #empty file
vendor/bin/codecept g:phpunit unit [CLASSNAME]
~~~

You can edit functional test class by adding one test function as following which similiar to `Gherkin` language.
~~~
public function indexActionTest(FunctionalTester $I)
{
  $I->wantTo('To see the welcome message on home page');
  $I->amOnPage('/');
  $I->see('Welcome');
}
~~~

For unit test it will following the basic of unit test. something like this.
~~~
public function testAboutAction()
{
    $this->assertTrue(true);
}
~~~

You can run test command as following.
It will output the result to tell whether pass or not.
~~~
vendor/bin/codecept run
vendor/bin/codecept run [suit]
vendor/bincodecept run [suit] [Test]
vendor/bin/codecept run [suit] —steps
~~~
---
[selenium2_download]: http://www.seleniumhq.org/download/
[geckodriver_download]: https://github.com/mozilla/geckodriver/releases
[chromedriver_download]: https://sites.google.com/a/chromium.org/chromedriver/downloads
[mink_doc]: http://mink.behat.org/en/latest/index.html
