---
title: Programming create Media Entity image Bundle content in D8
teaser: Utility / Content / D8
category: utility
tags: [utility]
---

Requirements: has a basic drupal project.
----------------------------------------


  ### 1. code is as fllowing and well self-exaplained:

    ```
      use Drupal\media\Entity\Media;
      use Drupal\file\Entity\File;

      function crain_config_updates_update_8157() {

        // using drupal stream_wrapper_manager service to resolve public address
        $image_data = file_get_contents(\Drupal::service('stream_wrapper_manager')->getViaUri('public://video_player_button.png')->getExternalUrl());

        // save image data to file.
        $file_image = file_save_data($image_data, 'public://video_player_button.png', FILE_EXISTS_REPLACE);

        // Create media entity with saved file.
          $image_media = Media::create([
            'bundle' => 'image',
            'uid' => \Drupal::currentUser()->id(),
            'status' => 1,
            'langcode' => \Drupal::languageManager()->getDefaultLanguage()->getId(),
            'image' => [
              'target_id' => $file_image->id(),
              'alt' => t('video_player_button'),
              'title' => t('video_player_button'),
            ],
          ]);
          $image_media->save();
        }
    ```

  ### 2. run following command to install image bundle content.

    ```
      drush updb -y # when user hook_update_N()
    ```

  > to synchronize content config to live.
