---
title: Serveral Useful function in D8
teaser: Utility / D8
category: utility
tags: [utility]
---

Requirements: has a basic drupal project.
----------------------------------------


  ### 1. Following describe serveral useful function for D8:

    ```
      // Where key can be 'node', 'taxonomy_term' which automatically DI entity for further user
      \Drupal::routeMatch()->getParameter($key);
    ```

    ```
      // Get request HTTP_REFERER, for example, ajax render to get actually page info.
      $previousUrl = \Drupal::request()->server->get('HTTP_REFERER');

      // Create a fake request based on HTTP_REFERE.
      $fakeRequest = \Symfony\Component\HttpFoundation\Request::create($previousUrl);
      // Get Path info.
      $path = $fakeRequest->getPathInfo();
      // Get Actually path from path alias, which can be useful for get render NID or TID.
      $rawPath = \Drupal::service('path.alias_manager')->getPathByAlias($path);
    ```
  ### 2. Following useful hook for view on D8:

  > for infinite view loading inorder to get the actually count for view item. can use hook_views_theme(&$variables).
  > Then additional values can passing to attributes item.
  > example as following:

  ```
  /**
  * Set Omni var from Node.
  *
  * @param array $variables
  *   Render array.
  * @param mixed $node
  *   Node var.
  */
  function _crain_story_components_set_omni_var_from_node(array &$variables, $node = NULL) {
    if (($node instanceof Node) && $node->getType() == 'section_pages') {
      $variables['options']['attributes']['data_omnilocation'] = 'section-' . str_replace(' ', '-', $node->getTitle());
    }
  }

  /**
  * Set Omni var from Term.
  *
  * @param array $variables
  *   Render array.
  * @param mixed $term
  *   Term var.
  */
  function _crain_story_components_set_omni_var_from_term(array &$variables, $term = NULL) {
    if (($term instanceof Term) && $term->bundle() == 'topics') {
      $variables['options']['attributes']['data_omnilocation'] = 'topic-' . str_replace(' ', '-', $term->getName());
    }
  }

  /**
  * Get ajax refer entity Info.
  *
  * @param mixed $node
  *   Node var.
  * @param mixed $term
  *   Term var.
  *
  * @return array
  *   Return var.
  */
  function _crain_story_components_get_refer_entity_info($node = NULL, $term = NULL) {
    if ($node == NULL && $term == NULL) {
      $previousUrl = \Drupal::request()->server->get('HTTP_REFERER');
      $fakeRequest = Request::create($previousUrl);
      $path = $fakeRequest->getPathInfo();
      $rawPath = \Drupal::service('path.alias_manager')->getPathByAlias($path);
      if (preg_match('/node\/(\d+)/', $rawPath, $matches)) {
        $node = Node::load($matches[1]);
      }
      if (preg_match('/taxonomy\/term\/(\d+)/', $rawPath, $matches)) {
        $term = Term::load($matches[1]);
      }
    }
    return [$node, $term];
  }

  /**
  * Implements hook_preprocess_views_infinite_scroll_pager().
  *
  * To add customize omniture tracking.
  */
  function crain_story_components_preprocess_views_infinite_scroll_pager(&$variables) {
    $node = \Drupal::routeMatch()->getParameter('node');
    $term = \Drupal::routeMatch()->getParameter('taxonomy_term');

    list($node, $term) = _crain_story_components_get_refer_entity_info($node, $term);
    _crain_story_components_set_omni_var_from_node($variables, $node);
    _crain_story_components_set_omni_var_from_term($variables, $term);
  }

  ```
