---
title: Install Drupal Entity Content Using Array
teaser: Utility / Drupal / Entity Content
category: utility
tags: [utility, workflow]
---

Requirements: has a basic drupal project
----------------------------------------

### config vs. entity content
  > config manager can allow us to export a lots of core config such as views.
  > however, if you want to export content from drupal to code, need the help of custom module or you can using entity content array

### prepare content and hook functions:

  1. first create an array store entity content based on property
    as following:

    ```
      $block_content = [
        'block_content' => [ # entity content type such as user, node
          'c3b66efd-b1eb-40e4-bc2e-17227bcdfd10' => [ # entity content uuid
            'original' => [
              'type' => 'section_story_component', # entity custom type such as 'basic' or custom type of different block and so on
              'info' => 'Update timestamp styles', # description
              'field_section_page_articles' => [
                1846, # short for ['target_id' => 1846] , which is the reference entity id
                2246,
                2261,
                2291,
                2296,
                2121,
              ],
              'field_topics' => [
                32186,
                2391,
                32181,
              ],
              'field_content_type' =>  'photo_gallery',
            ],
          ]
        ]
      ];
    ```

  2. you can create a custom function to save entity content as following:

  ```
    function crain_config_updates_install_profile_create_content(array $content) {
      foreach ($content as $entity_type_id => $items) {
        $storage = \Drupal::entityTypeManager()->getStorage($entity_type_id);
        foreach ($items as $uuid => $item) {
          $original = $item['original'];
          $entity = $storage->create($original + ['uuid' => $uuid]);
          $entity->save();
          if (array_key_exists('translations', $item)) {
            $translations = $item['translations'];
            foreach ($translations as $langcode => $content) {
              $translation = $entity->addTranslation($langcode, $content);
              $translation->save();
            }
          }
        }
      }
    }
  ```
  3. create a module install file and create hook_update_N() function or hook_install to install entity content

  ```
    function my_yaml_content_module_update_8003() {
      $block_content = ...;
      crain_config_updates_install_profile_create_content($block_content);
    }

  ```

  or

  ```
    function my_yaml_content_module_install() {
      $block_content = ...;
      crain_config_updates_install_profile_create_content($block_content);
    }
  ```

### import content config to live site:
  > after import the above two code block into one of the module install file
  > go to live site, run:

  ```
  drush updb -y # when user hook_update_N()
  ```

  or

  ```
    drush en module_name # when use hook_install()
  ```

  > to synchronize content config to live.
