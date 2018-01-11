---
title: Drupal8 Drush MAMP Environment
teaser: Environment / MAMP / Drupal8 / Drush
category: environment
tags: [environment, workflow]
---

Install Drupal8 using Drush on MAMP.

Requirements: Drush, MAMP
----------------------------------------

### MAMP
  Prepare MAMP:
    download MAMP and install on local machine
  Add MAMP mysql path to Enviornment $PATH to allow mysql can execute through command line:
  ~~~
    export PATH="/usr/local/sbin:/Applications/MAMP/Library/bin:$PATH"
  ~~~

  Create a database through terminal or other tools like MYSQL WorkBench as following:
  ~~~
    CREATE DATABASE database CHARSET utf8 COLLATE utf8_general_ci;
    CREATE USER username@localhost IDENTIFY BY 'pass';
    GRANT SELECT, INSERT, DELETE, UPDATE, DROP, CRATE, INDEX, ALERT, CREATE TEMPORY TABLE ON database.* TO 'username'@'localhost' INDENTIFY BY 'pass';
  ~~~

### Install drupal8
  > go to site root directory of MAMP

  run drush download drupal8:
  ~~~
    drush dl drupal-8
  ~~~
  
  > go to donwloaded drupal-8 directory

  run drush site install:
  ~~~
    drush si --locale=en --account-username=username --account-pass=pass --account-mail=username@example.com --db-url=msql://username:pass@127.0.0.1:8889/database --db-prefix=db-drupal-8
  ~~~
