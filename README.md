# proxy-build
vagrant configuration file for proxy vnf development environment. 

It deploys an ubuntu 14.04 server on which the Squid proxy and the dashboard will run.

## Host machine requirements

* Install
	* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
	* [Vagrant](http://docs.vagrantup.com/v2/installation/)
	* Install Vagrant plugins ``vagrant-useradd`` and ``vagrant-vbguest``
					      
			vagrant plugin install vagrant-useradd
			vagrant plugin install vagrant-vbguest
																			
	The ``vagrant-useradd`` plugin enables Vagrant to create users in the VM, before setting up file sharing.

	The ``vagrant-vbguest`` plugin makes sure that whenever we run ``vagrant up``, the correct version of ``VirtualBox Guest Additions`` will be installed.

## How to set up a development environment from scratch

#### Set up the environment and VM

1) Create the folder ``dev`` and cd into it

		mkdir dev
		cd dev

2) Clone the ``proxy-build`` repository, and cd into the folder	

		git clone git@github.com:dimosthe/proxy-build.git 
		cd proxy-build    

3) Create the folder ``shared``

		mkdir shared

This folder will expose some application files and folders to the host filesystem.

4) [Optional] If you want to change the VM's default IP, edit the ``Vagrantfile`` and modify the IP defined in line

		config.vm.network :private_network, ip: "192.168.56.120"

For the rest of the documentation, we'll assume that the IP is ``192.168.56.120``

5) Create and run the VM

		vagrant up

Once this command finishes, the VM is up and running. Our user is ``vagrant`` and password ``vagrant``. The VM is bound to the local IP ``192.168.56.120``.

6) Now you can use command ``vagrant ssh`` to SSH into the VM, without providing any password.

#### Deploy apache2 web server, MySql server and build the Dashboard application

1) Install necessary packages
	
	* apache2
	* mysql-server-5.5
	* git
	* curl
	* php5
	* php5-curl
	* php5-intl
	* php5-mcrypt
	* php5-mysql
	* php5-imagick
	* libapache2-mod-php5

2) [Generate SSH keys](https://help.github.com/articles/generating-ssh-keys/) for the user ``proxyvnf`` in order to use the ``Squid-dashboard`` repository. Firstly change to user ``proxyvnf`` 
	
	sudo -u proxyvnf -s 

and follow the instructions above

3) Clone the ``Squid-dashboard``
	
	cd /home/proxyvnf/dashboard
	git clone git@github.com:dimosthe/Squid-dashboard.git

4) Install Composer

	curl -s http://getcomposer.org/installer | php
	sudo mv composer.phar /usr/local/bin/composer

5) Run ``composer global require "fxp/composer-asset-plugin:1.0.0-beta3"``. Installs the composer asset plugin which allows managing bower and npm package dependencies through Composer. You only need to run this command once for all.

6) Run ``composer install`` in the root directory of the ``Squid-dashboard`` application in order to install dependencies. This will create the vendor directory with all package dependencies inlcuding the yii core source code.
	
7) Enable apache rewrite module
	
	sudo a2enmod rewrite
	sudo service apache2 restart
	
8) Create a symlink to point to the ``Squid-dashboard`` application

	cd /var/www/html
	sudo ln -s /home/proxyvnf/dashboard/Squid-dashboard/ dashboard

9) Set document root to be ``/var/www/html/dashboard/web``

	sudo vim /etc/apache2/sites-available/000-default.conf
	DocumentRoot "/var/www/html/dashboard/web" 

10) Hide ``index.php`` from the url

	sudo vim /etc/apache2/apache2.conf
	<Directory "/var/www/html/dashboard/web">
		# use mod_rewrite for pretty URL support
		RewriteEngine on
		# If a directory or a file exists, use the request directly
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteCond %{REQUEST_FILENAME} !-d
		# Otherwise forward the request to index.php
		RewriteRule . index.php

		# ...other settings...
	</Directory>

11) Test the application 

	http://192.168.56.120
