---
title: Multi-site set up for drupal for local development
teaser: Environment / Drupal / BLT/ Drupal VM
category: Environment
tags: [environment, workflow]
---

Requirements: has a basic drupal project
----------------------------------------

### acquia cloud multisite set up part:
  > create a new database for a new site

  > create a new site directory codebase under docroot/sites,
    > such as docroot/sites/chg

  > create sites.php file to direct incoming HTTP requests to appropriate site
   > such as:

  ```
    $sites = array(
      'drupal.dev.chicagobusiness.com' => 'chg',
      'drupal.stage.chicagobusiness.com' => 'chg',
      'drupal.prod.chicagobusiness.com' => 'chg',
    );
  ```
  > Add and require the site-specific settings (database credentials) include to each site's settings.php file, such as doctroot/sites/chg/settings.php, before the require statement for blt.settings.php, and sync directory

  ```
    if (file_exists('/var/www/site-php')) {
      require '/var/www/site-php/craind8dev/D8-dev-chg-settings.inc';
    }

    require DRUPAL_ROOT . "/../vendor/acquia/blt/settings/blt.settings.php";
    $config_directories[CONFIG_SYNC_DIRECTORY] = '../config/chg/sync';
  ```

### blt set up:
  > multisite codebase as following inside blt/project.yml:

  ```
  multisite:
    name:
      - default
      - sitename2 # which name corresponding to the folder name under docroot/sites
  ```

  > ensure new project has $settings['install_profile'] set up in docroot/sites/cng/settings.php file

### local set up:
  > step 1: download multi-site database from acquia cloude

  > step 2: find corresponding require file from doctroot/sites/chg/settings.php and download them, import into your local drupal vm, also if there is not additional setting need work with your local multisite you can skip 2, go to step 3

  > step 3: if you have done step 2, please skip it. otherwise, please copy doctroot/sites/chg/settings/default.local.settings.php to doctroot/sites/chg/settings/local.settings.php, and update the file with your own local database info for site chg.

  > step 4: you can choose to directory modify box/config.yml file

  > OR

  > copy box/config.yml file to box/local.config.yml file and set up local multi-site as following to set multi-site support for drupal vm:

  {% raw %}
    # Other configuration.
    dashboard_install_dir: "/var/www/dashboard"


    apache_vhosts:
      # Drupal VM's default domain, evaluating to whatever `vagrant_hostname` is set to (drupalvm.test by default).
      - servername: "local.drupal.dev.autonews.com"
        documentroot: "{{ drupal_core_path }}"
        extra_parameters: "{{ apache_vhost_php_fpm_parameters }}"

      - servername: "drupal.dev.chicagobusiness.com"
        documentroot: "{{ drupal_core_path }}"
        extra_parameters: "{{ apache_vhost_php_fpm_parameters }}"

      - servername: "{{ vagrant_ip }}"
        serveralias: "dashboard.{{ vagrant_hostname }}"
        documentroot: "{{ dashboard_install_dir }}"

      - servername: "adminer.{{ vagrant_hostname }}"
        documentroot: "{{ adminer_install_dir }}"
        extra_parameters: |
          ProxyPassMatch ^/(.*\.php(/.*)?)$ "fcgi://127.0.0.1:9000{{ adminer_install_dir }}"

      - servername: "xhprof.{{ vagrant_hostname }}"
        documentroot: "{{ php_xhprof_html_dir }}"
        extra_parameters: |
              ProxyPassMatch ^/(.*\.php(/.*)?)$ "fcgi://127.0.0.1:900{*{ php_xhprof_html_dir }}"

      - servername: "pimpmylog.{{ vagrant_hostname }}"
        documentroot: "{{ pimpmylog_install_dir }}"
        extra_parameters: |
              ProxyPassMatch ^/(.*\.php(/.*)?)$ "fcgi://127.0.0.1:900{*{ pimpmylog_install_dir }}"


    apache_packages_state: latest
    apache_remove_default_vhost: true
    apache_mods_enabled:
      - expires.load
      - headers.load
      - ssl.load
      - rewrite.load
      - proxy.load
      - proxy_fcgi.load

    # Nginx hosts. Each site will get a server entry using the configuration defined
    # here. Set the 'is_php' property for document roots that contain PHP apps like
    # Drupal.
    # nginx_hosts:
    #   - server_name: "local.drupal.dev.autonews.com"
    #     root: "{{ drupal_core_path }}"
    #     is_php: true

    #   - server_name: "drupal.dev.chicagobusiness.com"
    #     root: "{{ drupal_core_path }}"
    #     is_php: true

    #   - server_name: "adminer.{{ vagrant_hostname }}"
    #     root: "{{ adminer_install_dir }}"
    #     is_php: true

    #   - server_name: "xhprof.{{ vagrant_hostname }}"
    #     root: "{{ php_xhprof_html_dir }}"
    #     is_php: true

    #   - server_name: "pimpmylog.{{ vagrant_hostname }}"
    #     root: "{{ pimpmylog_install_dir }}"
    #     is_php: true

    #   - server_name: "{{ vagrant_ip }} dashboard.{{ vagrant_hostname }}"
    #     root: "{{ dashboard_install_dir }}"
    #     is_php: true

    # nginx_remove_default_vhost: true
    # nginx_ppa_use: true
    # nginx_vhost_template: "templates/nginx-vhost.conf.j2"



    # MySQL Databases and users. If build_makefile: is true, first database will
    # be used for the makefile-built site.
    drupal_mysql_database: "drupal"
    drupal_chg_mysql_database: "drupal_chg"
    mysql_databases:
      - name: "{{ drupal_mysql_database }}"
        encoding: utf8
        collation: utf8_general_ci

      - name: "{{ drupal_chg_mysql_database }}"
        encoding: utf8
        collation: utf8_general_ci

    mysql_users:
      - name: "{{ drupal_mysql_database }}"
        host: "%"
        password: "{{ drupal_mysql_database }}"
        priv: "{{ drupal_mysql_database }}.*:ALL"

      - name: "{{ drupal_chg_mysql_database }}"
        host: "%"
        password: "{{ drupal_chg_mysql_database }}"
        priv: "{{ drupal_chg_mysql_database }}.*:ALL"
  {% endraw %}

  run
  ```
    vagrant provision
  ```
  or
  ```
    vagrant reload --provision
  ```
  > step 5: create a new database corresponding to required database info set up in doctroot/sites/chg/settings.php, which can be found in the additional required file, such as /var/www/site-php/craind8dev/D8-dev-chg-settings.inc (step2) or doctroot/sites/chg/settings/local.settings.php (step3) . Remember to change it remote ip address to your local drupal vm private ip address or localhost.

  > step 6: import download databases to multi-site, chg database.

  > step 7: visit your local site for multi-site like: drupal.dev.autonews.com, drupal.dev.chicagobusiness.com. The local multi-site environment should set up now.

---
  useful settings for update drupal vm php settings:
    if database import files is to smaller you can change box/local.config.yml file php ini as following:

    ```
      php_memory_limit: "512M"
      php_post_max_size: "512M"
      php_upload_max_filesize: "512M"
    ```

  useful settings for run drush @site command for specific in multisite:
    edit drush/site-aliases/aliases.drushrc.php file or add a new file called "drush/site-aliases/local.aliases.drushrc.php" by adding following site alias porition:

    ```
    // an environment.
    $aliases['drupal.dev.autonews.local'] = array(
      'root' => '/var/www/crain-platform/docroot',
      'uri' => 'http://local.drupal.dev.autonews.com',
      );
    // Add remote connection options when alias is used outside VM.
    if ('vagrant' != $_SERVER['USER']) {
      $aliases['drupal.dev.autonews.local'] += array(
        'remote-host' => 'local.drupal.dev.autonews.com',
        'remote-user' => 'vagrant',
        'ssh-options' => '-o PasswordAuthentication=no -i ' . drush_server_home() . '/.vagrant.d/insecure_private_key'
      );
    }


    // chg environment.
    $aliases['drupal.dev.chicagobusiness.local'] = array(
      'root' => '/var/www/crain-platform/docroot',
      'uri' => 'http://drupal.dev.chicagobusiness.com',
      );
    // Add remote connection options when alias is used outside VM.
    if ('vagrant' != $_SERVER['USER']) {
      $aliases['drupal.dev.chicagobusiness.local'] += array(
        'remote-host' => 'drupal.dev.chicagobusiness.com',
        'remote-user' => 'vagrant',
        'ssh-options' => '-o PasswordAuthentication=no -i ' . drush_server_home() . '/.vagrant.d/insecure_private_key'
      );
    }
  ```

  Then you can run following example commands as convenience:

  ```
    drush @drupal.dev.chicagobusiness.local uli
    drush @drupal.dev.chicagobusiness.local status
    drush @drupal.dev.autonews.local uli
    drush @drupal.dev.autonews.local status
  ```
