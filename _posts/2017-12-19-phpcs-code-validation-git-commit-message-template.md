---
title: BLT code validation and Git commit message template
teaser: Environment / PHP Code Sniffer/ Git
category: Environment
tags: [environment, workflow]
---
BLT code validation

Requirements: code project configured using BLT installed Drupal VM
----------------------------------------

### prepare environment
  > copyp phpcs.xml.dist to phpcs.xml under project root
  > edit phpcs.xml file to configure which path or file that you want to add for code validation and which to exclude.
  > as following:

```
  <file>included files or directory</file>
  <exclude-pattern>exclude files or directory</exclude-pattern>
```

  > Then every time when you run git commit msg or run following commands, it will start validate your code.

  ```
    blt validate:all
    blt validate:phpcs
    blt validate:twig
    blt validate:yml
  ```



#### Git commit message use template:
  > create a txt file as git commit message template under repository root.
  > run git commit as following:

  ```
    git commit -t file_path_of_git_commit_msg_txt
  ```

  > if you want to config a global git commit message template, you can also do it by using:

  ```
    git config commit.template file_path_of_git_commit_msg_txt
  ```

  > Then everything time when you run a git commit commands. it will automatically use the template file to format git commit message.
