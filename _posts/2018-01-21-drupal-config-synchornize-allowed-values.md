---
title: Field Storage Synchornized in D8 with Allowed Values
teaser: Utility / Field Storage / D8
category: utility
tags: [utility]
---

Requirements: has a basic drupal project
----------------------------------------

### synchronized field storage with allowed values
  > if want to synchronized field storage with allowed values, like List(Text), List(Integer) etc. Facing issue with "The configuration property settings.allowed_values.0.label.0 doesn't exist". Can fix by following:


  1. remove all these allowed_values in the field storage to become an array as following:


    ```
      settings:
        allowed_values:
          -
            value: 1
            label: Ascend
          -
            value: 5
            label: Descend
        allowed_values_function: ''
    ```


  to


    ```
      settings:
        allowed_values: {  }
        allowed_values_function: ''
    ```


  and


    ```
      $allowed_values = [
        1 => 'Ascend',
        2 => 'Descend',
      ];
    ```

  2. Then can synchronized this field storage as following:

    ```
      function my_module_update_8003() {
        $config_path = file_get_contents(path_to_field_storage_yaml_file);
        $data = Yaml::decode($config_path);
        $fieldStorage = FieldStorageConfig::create($data);
        $fieldStorage->setSetting('allowed_values', [
          1 => 'Ascend',
          2 => 'Descend',
        ]);
        $fieldStorage->save();
      }

    ```

  3. run following command to install field storage config.

    ```
      drush updb -y # when user hook_update_N()
    ```

  > to synchronize content config to live.
