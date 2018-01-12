---
title: Utility of Yaml Content contribute module
teaser: Utility / Drupal / Yaml Content
category: utility
tags: [utility, workflow]
---

Requirements: has a basic drupal project
----------------------------------------

### yaml_content module vs drupal core config manager
  > config manager can allow us to export a lots of core config such as views.
  > however, if you want to export content from drupal to code, need the help of default_content or yaml_content module
  > yaml_content module all back end, doesn't have export function, but using the simplicity of yaml file allow create yaml file to synchronize between each site

### utility content module vs drupal core config manager
  Here introduce yaml module:
    first download and enable default content module
  ```
    composer require drupal/yaml_content
    drush en yaml_content
  ```
  
  1. first create an custom module or profile
  2. create a directory named as content
  3. add content yaml file into content directory
  e.g. xxx_block.content.yml
  ```
    - entity: "block_content"
      type: "navigation_banner"
      info: "Test Default Banner Update"
      id: 2
      uuid: "867753c9-655b-4e34-969b-47ea4b2077bc"
      field_banner_link:
        - value: "http://google.com"
      field_banner_text:
        - value: "test default banner"
  ```
  4. create a module install file and create hook_update_N() function
  ```
    function my_yaml_content_module_update_8003() {
      $my_yaml_content_module = 'my_yaml_content_module';
      $path = drupal_get_path('module', $my_yaml_content_module);
      $loader = \Drupal::service('yaml_content.content_loader');
      $loader->setContentPath($path);

      $content_files = [
        'xxx_block.content.yml',
      ];

      $load_entities = [];
      foreach ($content_files as $file_name) {
        $loaded = $loader->loadContent($file_name, $update);
        $load_entities = array_merge($load_entities, $loaded);
      }
    }

  ```

### import content config to live site:
  > after import the above two code block into one of the module install file
  > go to live site, run:

  ```
  drush updb -y
  ```

  > to synchronize content config to live.

  useful commands:
  ```
    drush yaml-content-import-module <module_name>
  ```
