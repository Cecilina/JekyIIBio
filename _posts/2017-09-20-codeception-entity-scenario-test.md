---
title: Codeception Scenario
teaser: Functional / Unit / Acceptance Codeception Test
category: test
tags: [Test, Workflow]
---

Codeception Scenario Test for BDD and TDD on Symfony3.

Requirements: Composer
----------------------------------------

### ERDs and ORM
 Reverse Engineer create ERDs (Entity Relationship Diagrams) created in MySQL Workbench:
    `cmd + R` or `ctrl + R` or Database > Reverse Engineer
 Forward Engineer create tables in database based on ERDs
    `cmd + G` or `ctrl + G` or Database > Forward Engineer
 generate ORM (Object Relation Mapping) based on database
  ~~~
    bin/console doctrine:mapping:import --force [BundleName] [type] # type is configuration file type. e.g. `yml`
    bin/console doctrine:mapping:convert annotation [directory] #use annotation to generate Entity
  ~~~
  check whether these Entity class file has annotation of repositoryClass for `@ORM\Entity`.
  if not, it won't generate get and set method add them, they run
  ~~~
    bin/console doctrine:generate:entities [BundleName]
  ~~~

### Create tester Data using `Data Fixture`
  >### Notice getOrder function return the order which fixture will be loaded.
  > if there is an relationship between each entity, remember to return the correct Integer for this function
  > smaller, loaded earlier
  run commands
  ~~~
    bin/console doctrine:fixture:load # add data to database
  ~~~

### Configure each test suit

functional suit: add Symfony framework
~~~
      - Symfony:
        app_path: 'app'
        var_path: 'var'
        environment: 'test'
~~~

Unit suit:
~~~
- Asserts
- \Helper\Unit
- Db
- Symfony:
    # app_path: 'app' #disable these settings when run all functional test and unit tester together e.g.  vendor/bin/codecept run --steps --debug
    # var_path: 'var'
    # environment: 'test'
- Doctrine2:
    depends: Symfony
~~~

Acceptance suit:
~~~
- WebDriver:
  url: http://localhost:8000
  browser: firefox
- \Helper\Acceptance
~~~

use codecept generate test class for each suit:
~~~
vendor/bin/codecept g:test unit [CLASSNAME] # more function than phpunit
vendor/bin/codecept g:cest functional [CLASSNAME]
vendor/bin/codecept g:cest acceptance [CLASSNAME]
~~~

run test individually or together as following:
~~~
vendor/bin/codecept suit [CLASSNAME]
vendor/bin/codecept run
vendor/bin/codecept run --steps --debug
~~~

>#### Notice:
> 1. when run acceptance test, remember need start selenium2 server for browser test
> 2. if run together remember avoid redefine variables for each framework

---
