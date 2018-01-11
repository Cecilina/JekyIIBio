---
title: Utility of Default Content contribute module
teaser: Utility / Drupal / Default Content
category: Utility
tags: [Utility, workflow]
---

Requirements: has a basic drupal project
----------------------------------------

### default content module vs drupal core config manager
  > config manager can allow us to export a lots of core config such as views.
  > however, if you want to export content from drupal to code, need the help of default_content or yml_content module

### utility content module vs drupal core config manager
  Here introduce default content:
    first download and enable default content module
  ```
    drush pm-download default_content
    drush en default_content
  ```
  for example when use

  ```
    drush dce block_content block_id --file=file_path
  ```

  then you can use hook_update_N() function to import local environment content config to live site
  as following: support using content_type as folder and content uuid as filename to write import content script as following:

  ```
  /**
  * update content config from json
  *
  * @param string $path_to_content_json
  *  $path_to_content_json following follow rules: content_type/uuid
  */
  function crain_config_updates_json($path_to_content_json) {
    list($entity_type_id, $filename) = explode('/', $path_to_content_json);
    $path_profile = drupal_get_path('profile', 'profile_name');
    $encoded_content = file_get_contents($path_profile . '/config/install/' . $path_to_content_json . '.json');
    $serializer = \Drupal::service('serializer');
    $content = $serializer->decode($encoded_content, 'hal_json');
    global $base_url;
    $url = $base_url . base_path();
    $content['_links']['type']['href'] = str_replace('http://drupal.org/', $url, $content['_links']['type']['href']);
    $contents = $serializer->encode($content, 'hal_json');
    $class = 'Drupal\\' . $entity_type_id . '\Entity\\' . str_replace(' ', '', ucwords(str_replace('_', ' ', $entity_type_id)));
    $entity = $serializer->deserialize($contents, $class, 'hal_json', array('request_method' => 'POST'));
    $entity->enforceIsNew(TRUE);
    $entity->save();
  }
  ```

  ```
  /**
  * Implements hook_update_N().
  *
  */
  function crain_config_updates_update_8128() {
    $jsons = [
      'node/dde43db4-b2dc-4baf-86c0-68a468e5effb',
      'block_content/c3b66efd-b1eb-40e4-bc2e-17227bcdfd10',
    ];
    foreach ($jsons as $json) {
      crain_config_updates_json($json);
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
    drush dcem my_default_content_module
    drush dce node <node id>
    drush dce taxonomy_term <taxonomy term id>
    drush dce file <file id>
    drush dce media <media id>
    drush dce menu_link_content <menu link id>
    drush dce block_content <block id>
    drush dce taxonomy_term <tid>
    drush dce node <node id> --folder=folder_name

  ```
