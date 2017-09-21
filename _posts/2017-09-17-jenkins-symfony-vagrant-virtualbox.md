---
title: Continuous Integration (CI)
teaser: Continuous Integration for Symfony (PHP7.0) using Jenkins in Vagrant Virtualbox
category: env
tags: [environment, workflow, deploy]
---

Set up an Continuous Integration (CI) Environment for Symfony (php7.0) in Vagrant Virtualbox in local vitrual machine.

Requirements: Vagrant, Virtualbox

Environment: <b>host:</b> `Mac` <b>virtual:</b> `Linux`

Install `Linux` Virtual Machine managed by `Vagrant`
----------------------------------------

### 1.1 Install `Linux` Virtual Machine chosen Virtualbox

Here use `brew` to install virtualbox and its extension
{% highlight shell %}
brew cask install virtualbox
brew cask install virtualbox-extension-pack
{% endhighlight %}

### 1.2 Install `Vagrant` to manage virtual machine

{% highlight shell %}
brew cask install vagrant
brew cask install vagrant-manager
{% endhighlight %}

### 1.3 Initialize a virtial box since we need PHP7.0 choose Vagrant box `ubuntu/xenial64`

You can click [here][boxes] to choose different vagrant boxes.

> #### Note: each vagrant virtual machine need have a box

You choose one of the following method to Initialize and Linux virtual machine

> #### approach 1: set up a default virtual machine using command
{% highlight shell%}
vagrant init ubuntu/xenial64
vagrant up
vagrant ssh
{% endhighlight %}

> #### approach 2: use config file to set an virtual machine named as Vagrantfile
<b>config file as following: </b>
{% highlight vagrant %}
   Vagrant.configure("2") do |config|
     config.vm.box = "ubuntu/xenial64"
     config.vm.define "symfony", primary: true, autostart: false do |symfony|
       config.vm.network "private_network", ip: "192.168.33.10"
     end
   end
{% endhighlight %}
> #### <b>Then run command as following</b>
> for login virtual machine password, please refer to use vagarnt boxes virtial Vagrantfile file, usually in following fold
> `{% raw %} ~/.vagrant.d/boxes/boxname/Vagrantfile {% endraw %}`
{% highlight shell%}
vagrant up symfony
sudo ssh ubuntu@192.168.33.10
{% endhighlight %}


Prepare Virtual Machine Environment
--------------------------------------
here I install nginx as server. you can also choose apache2
~~~
sudo apt-get upgrade && sudo apt-get update
sudo apt-get install language-pack-en
sudo apt-get install nginx
sudo apt-get install php7.0 php7.0-mysql mysql-client mysql-server php-mbstring php-xml php7.0-gd
~~~

As jenkins need java 8, so first install java
~~~
sudo apt-get install openjdk-8-jre
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get install jenkins
~~~

[^1]Configure jenkins server as reverser proxy for nginx. please append following to nginx default
~~~
# Note that regex takes precedence, so use of "^~" ensures earlier evaluation
  location ^~ /jenkins/ {

      # Convert inbound WAN requests for https://domain.tld/jenkins/ to
      # local network requests for http://192.168.33.10:8080/jenkins/
        proxy_pass http://192.168.33.10:8080/jenkins/;

      # Rewrite HTTPS requests from WAN to HTTP requests on LAN
        proxy_redirect http:// https://;

      # The following settings from https://wiki.jenkins-ci.org/display/JENKINS/Running+Hudson+behind+Nginx
        sendfile off;

        proxy_set_header   Host             $host:$server_port;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_max_temp_file_size 0;

        #this is the maximum upload size
        client_max_body_size       10m;
        client_body_buffer_size    128k;

        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;

        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;

        # Required for new HTTP-based CLI
        proxy_http_version 1.1;
        proxy_request_buffering off;
      }
~~~

using following command to check whether nginx config is correct or not
~~~
nginx -t
~~~

As we prefix jenkins url with jenkins., please edit jenkins config file. e.g. /etc/default/jenkins by added prefix
~~~
JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --prefix=/jenkins"
~~~

reload jenkins and nginx server as following
~~~
sudo service jenkins reload
sudo service nginx reload
sudo service jenkins restart
sudo service nginx restart
~~~

Config Jenkins
-------------------------------------------------------
now go to http://192.168.33.10:8080/jenkins/[jenkins_url] to config jenkins.
It will initially ask for password, which you can find it at it log file, usually here
~~~
/var/log/jenkins/jenkins.log
~~~

config jenkins global security as following:
![Jenkins Global Security]({{ site.baseurl }}/i/Configure_Global_Security_Jenkins.png)

Install following plugins for jenkins:
> #### plugins
> Bitbucket Plugin; Checkstyle Plug-in; Clover PHP plugin; Crap4J plugin; DRY Plug-in; Email Extension Plugin; Email Extension Template Plugin
> HTML Publisher plugin; JDepend Plugin; Plot plugin; PMD Plug-in; Violations plugin; xUnit plugin

> #### [^2]composer install dependencies for jenkins php project
{% highlight shell %}
sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
sudo chown -R $USER:$USER ~/.composer/
sudo chown -R $USER:$USER ~/.composer/*
composer global config minimum-stability dev
composer global config prefer-stable true
composer global require phpunit/phpunit squizlabs/php_codesniffer \
    phploc/phploc pdepend/pdepend phpmd/phpmd sebastian/phpcpd \
    mayflower/php-codebrowser theseer/phpdox:dev-master
#make code style using symfony
composer require --dev escapestudios/symfony2-coding-standard:3.x-dev
vendor/bin/phpcs --config-set installed_paths vendor/escapestudios/symfony2-coding-standard
vendor/bin/phpcs -i
sudo apt-get install ant
{% endhighlight %}

Configure Jenkins global Environment variable path as following: since we use composer to install all these dependencies
![Jenkins Configure1]({{ site.baseurl }}/i/Configure_System_Jenkins_1.png)
Configure Email Notify as following using gmail[^3].
![Jenkins Configure2]({{ site.baseurl }}/i/Configure_System_Jenkins_2.png)


Create an Jenkins deploy item
------------------------------------------
Create a new Item as following:
![Jenkins Item1]({{ site.baseurl }}/i/Configure_Item_Jenkins_1.png)[^4]
![Jenkins Item2]({{ site.baseurl }}/i/Configure_Item_Jenkins_2.png)

Build it first
-------------------------------
Click `Build Now` to build. You can Click `Console Output` to check output.
![Jenkins build1]({{ site.baseurl }}/i/build_Jenkins_1.png)
Also you can dashboard.
![Jenkins build2]({{ site.baseurl }}/i/build_Jenkins_2.png)


---
[^1]:
    Assume that jenkins listen at 8080 port.

[^2]:
    If system throw `composer cannot allocate memory`. Please allocate it as following:
    ~~~
    sudo fallocate -l 1G /swapfile #create an empty 1GB file
    sudo mkswap /swapfile #format it as swap space
    sudo swapon /swapfile #enable this space so that the kernel begins to use it
    ~~~

[^3]:
    As notify you need set up gmail account for `gmail less secure apps`

[^4]:
    You will enter permission denied to get to your git repository. Please using following commands to generate ssh key and get access to it
    ~~~
    sudo su - jenkins
    ssh-keygen
    git ls-remote -h git_repository HEAD
    ~~~

[boxes]: https://app.vagrantup.com/boxes/search
[jenkins_url]: http://192.168.33.10:8080/jenkins/
