---
title: Blt Multisite Sync in D8
teaser: Utility / D8 / BLT(8.x)
category: utility
tags: [utility]
---

Requirements: has a basic drupal project.
----------------------------------------


  ### 1. Please following previous blog [Multi-site set up for drupal for local development](https://cecilina.github.io/JekyIIBio/posts/2018-01-02-blt-drupal-vm-multisite-set-up)

  ### 2. Following useful hook for view on D8:

  > In drush folder create two drush aliases file. one for local site alias, one for remote.

  remote as following (e.g.remote.aliases.drushrc.php): it well explained by itself to set up drush aliase corresponding to remote portional.
  ```
  $aliases['dev.aac'] = array(
    'root' => '/var/www/html/xxx/docroot',
    'ac-site' => 'xxx',
    'ac-env' => 'xxx',
    'ac-realm' => 'xxxx',
    'uri' => 'drupal.xxxxx.com',
    'remote-host' => 'xxxxx.xxxx.com',
    'remote-user' => 'xxxx.xxxx',

    'path-aliases' => array(
      '%drush-script' => 'drush' . $drush_major_version,
    ),
  );
  ```

  local as following (e.g.local.aliases.drushrc.php): it well explained by itself to set up drush aliase corresponding to remote portional.

  ```
  $aliases['aac'] = array(
    'uri' => 'aac',
    'root' => '/var/www/xxxx/docroot',
    'remote-host' => 'drupal.local.xxxx.com',
    'remote-user' => 'vagrant',
    'ssh-options' => '-o PasswordAuthentication=no -i ' . drush_server_home() . '/.vagrant.d/insecure_private_key'
  );
  ```

  ### 3. Under your multiple sites directory find the corresponding site folder(e.g. sites/aac):
  create a site.yml file as following:
  ```
  drush:
  aliases:
    local: local.dev.aac
    remote: remote.aac
  ```

  ### 4. Run following commands for sync:
  ```
    blt sync:db -D site=aac
    blt sync -D sit=aac
    blt sync:db:all # for all site database
  ```
