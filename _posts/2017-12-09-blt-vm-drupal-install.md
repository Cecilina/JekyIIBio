---
title: BLT Drupal VM install on Mac
teaser: Environment / BLT / Drupal8 / Drupal VM
category: Environment
tags: [environment, workflow]
---

Generate a Drupal  project on Drupal VM virtual machine through BLT (Build and Lanuch Tool).

Requirements: Git, xcode, Composer (global installed), Homebrew, PHP7.1, Vagrant, Virtualbox, Drush
----------------------------------------

### prepare environment
  install xcode:

  ```
    sudo xcodebuild -license
    xcode-select --install
  ```

  install Homebrew, php71, git, composer drush:

  ```
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    brew tap homebrew/dupes
    brew tap homebrew/versions
    brew tap homebrew/homebrew-php
    brew install php71 git composer drush
    composer global require "hirak/prestissimo:^0.3"
  ```

  install Virtualbox, Vagrant:

  ```
    brew tap caskroom/cask
    brew cask install virtualbox vagrant
    vagrant plugin install vagrant-hostsupdater
  ```

  change php environment by set memory_limit = 2G to allowed BLT to initialize:

  ```
    open -a /Applications/Visual\ Studio\ Code.app $(php -i | grep "Loaded Configuration File" | cut -d" " -f 5)
  ```

  for executing Behat test, install Java and test browser driver:

  ```
    brew cask install java
    brew install chromedriver
  ```

### Create a new project with BLT
  create a drupal project with BLT using composer

  ```
    composer clear-cache
    export COMPOSER_PROCESS_TIMEOUT=2000
    composer create-project --no-interaction acquia/blt-project projectname
  ```

  go to project directory
  ```
    composer run-script blt-alias
    source ~/.bash_profile
  ```
  if you are using zsh copy and paste the code added to .bash_profile to .zshrc, then 
  ```
    source ~/.zshrc
  ```
  
  edit the project.yml file to choose profile, if plan to use lighting profile. Please make sure git version and pacth version match. Otherwise, it will probably throw error when **blt setup**. suggest to update them all to the newest version.

  Additional notice check host php version, since current install php version is php7.1. Please change **vendor/acquia/blt/scripts/drupal-vm/config.yml**. php version to 7.1 to ensure drupal vm install the correct php version.

  create an drupal VM instance:
  ```
    blt vm
  ```
  check drupal VM setting can by visit **dashborad.local.projectname.com**
  install drupal project and setting:
  ```
    blt setup
  ```
  after install copy the drush alias file from drush/site-aliases/aliases.drushrc.php to ~/.drush/projectname.aliases.drushrc.php
  ```
    cp drush/site-aliases/aliases.drushrc.php ~/.drush/projectname.aliases.drushrc.php
  ```
  login to created drupal project by
  ```
    blt @projectname.local uli
  ```

  useful commands:
  ```
    vagrant reload
    vagrant global-status --prune
    composer self-update
    composer update acquia/blt --with-dependencies
    blt vm:nuke 
    blt validate
    blt tests:phpunit
    blt tests:behat
    blt doctor
    blt deploy:build
    blt deploy
    blt sync:refresh
    blt setup:update
    blt -l
  ```

<!-- 
      git -- version
      patch --version
      brew install git
      brew install gpatch
      brew link --overwrite git
      if you accidentally delete drupal vm, but drupal vm still in running statu, you can go to /etc to delete the added line for exports file or just delete the whole file.
-->