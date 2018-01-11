---
title: VSCODE XDEBUG for Drupal on Mac
teaser: Environment / VSCODE / XDEBUG / Drupal VM
category: environment
tags: [environment, workflow]
---

Config XDEBUG for Drupal debug on VSCODE.

Requirements: VSCODE, XDEBUG, Drupal Project, Vagrant, Virtualbox
----------------------------------------

### prepare environment

##### install VSCODE:

  > VSCODE can download and install from [here][vscode_download]:
  > Download and enable PHP Debug from VSCODE extensionsion marketplace.

  

#### Prepare a Drupal development virtual environment:

   > Method 1: Install Drupal VM virtual machine through Drupal VM [here][drupalVM_repository]
   > OR
   > Method 2: following [here][blt_install] to install and virtual machine using BLT

#### enable xdebug for virtual machine:
   
###### Method 1:
  > if you install drupal vm through Method1, please edit the config.yml file which in the root directory of git hub respository that you download as following for XDebug configuration:

  ```
  # XDebug configuration.
  # Change this value to 1 in order to enable xdebug by default.
  php_xdebug_default_enable: 1
  php_xdebug_coverage_enable: 0
  # Change this value to 1 in order to enable xdebug on the cli.
  php_xdebug_cli_enable: 1
  php_xdebug_remote_enable: 1
  php_xdebug_remote_connect_back: 1
  # Use PHPSTORM for PHPStorm, sublime.xdebug for Sublime Text.
  php_xdebug_idekey: VSCODE
  php_xdebug_max_nesting_level: 256
  php_xdebug_remote_port: "9000" # if you use nginx server change it to 9001 to prevent conflict
  php_xdebug_remote_host: "{{ anisible_default_ipv4.gateway }}"
  ```

###### Method 2:
  > if you install through BLT, edit box/config.yml file as following:

  ```
  # XDebug configuration.
  # Change this value to 1 in order to enable xdebug by default.
  php_xdebug_default_enable: 1
  php_xdebug_coverage_enable: 0
  # Change this value to 1 in order to enable xdebug on the cli.
  php_xdebug_cli_enable: 1
  php_xdebug_remote_enable: 1
  php_xdebug_remote_connect_back: 1
  # Use PHPSTORM for PHPStorm, sublime.xdebug for Sublime Text.
  php_xdebug_idekey: VSCODE
  php_xdebug_max_nesting_level: 256
  php_xdebug_remote_port: "9000" # if you use nginx server change it to 9001 to prevent conflict
  php_memory_limit: "512M"
  ```
  
### provision virtual machine
   > if you have already install the virtual machine, run

  ```
    vagrant provision
  ```

  to provision the virtual machine that you have installed


### install chrome xdebug extension for chrome 
  [here][chrome_xdebug_extension], you can also find other corresponding xdebug extension for different browser. and enable it to start debug

### add xdebug setting for xdebug
  > click debug button from right side of vscode , add a PHP xdebug lanuch.json as following:

  ```
    {
    "version": "0.2.0",
    "configurations": [
      {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "port": 9000,
        "pathMappings": {
          "server address": "local sync address" // suggest use absolute path
        }
      },
      {
        "name": "Launch currently open script",
        "type": "php",
        "request": "launch",
        "program": "${file}",
        "cwd": "${fileDirname}",
        "port": 9000
      }
    ]
  }
  ```

  select "Listen for XDebug", click start button or f5 to start debug.

  if you encouter any error, try to set up xdebug.remote_log file , default it should have already set up and default file to keep the log. you can through php -ini on VM or through phpinfo() to get these info.
  

---
[vscode_download]: https://code.visualstudio.com/
[drupalVM_repository]: https://github.com/geerlingguy/drupal-vm
[blt_install]: https://cecilina.github.io/JekyIIBio/posts/2017-12-09-blt-vm-drupal-install
[chrome_xdebug_extension]: https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en