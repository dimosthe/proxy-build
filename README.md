# ProXy as a Service (PXaaS) Virtual Network Function 
Vagrant configuration file for PXaaS vnf development environment for [T-NOVA](http://t-nova.eu/) project.  

It deploys an ubuntu 14.04 server and builds a [Squid proxy](http://www.squid-cache.org/), [SquidGuard](http://www.squidguard.org/index.html) and a [Dashboard](https://github.com/dimosthe/Squid-dashboard) on it.

The idea behind PXaaS vnf is to enables a user, who acts as the network administrator of his LAN, to configure the Squid Proxy on demand and provide Proxy services such as web caching, web access control, website filtering and user anonymity to the LAN's users. 

## Host machine requirements

* Install
	* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
	* [Vagrant](http://docs.vagrantup.com/v2/installation/)
	* Install Vagrant plugins ``vagrant-useradd`` and ``vagrant-vbguest``
					      
			sudo vagrant plugin install vagrant-useradd
			sudo vagrant plugin install vagrant-vbguest
																			
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

4) Create a new database and a new user and the yii2-user module

	mysql -u root -p
	create database dashboarddb
	create user 'dashboarduser'@'localhost' identified by 'primetel';
	grant all privileges on dashboarddb.* to dashboarduser@localhost;
	vim config/db.php // edit the file accordingly

5) Install Composer

	curl -s http://getcomposer.org/installer | php
	sudo mv composer.phar /usr/local/bin/composer

6) Run ``composer global require "fxp/composer-asset-plugin:~1.1.0"``. Installs the composer asset plugin which allows managing bower and npm package dependencies through Composer. You only need to run this command once for all.

7) Run ``composer install`` in the root directory of the ``Squid-dashboard`` application in order to install dependencies. This will create the vendor directory with all package dependencies inlcuding the yii core source code.

8) Install Squid 3.5.5 and SquidGuard (see intructions at the end of the page)

9) Install migrations. Run the following commands from the application's root directory

	sudo -u proxyvnf -s
	php yii migrate/up --migrationPath=@vendor/dektrium/yii2-user/migrations // in order to build the tables for the yii2-user module
	./yii createusers/create // from the root of the application. It creates a default user with username:admin, pass:administrator
	./yii migrate // to install other migrations
	
10) Enable apache rewrite module
	
	sudo a2enmod rewrite
	sudo service apache2 restart
	
11) Create a symlink to point to the ``Squid-dashboard`` application

	cd /var/www/html
	sudo ln -s /home/proxyvnf/dashboard/Squid-dashboard/ dashboard

12) Set document root to be ``/var/www/html/dashboard/web``

	sudo vim /etc/apache2/sites-available/000-default.conf
	DocumentRoot "/var/www/html/dashboard/web" 

13) Hide ``index.php`` from the url

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

14) Allow apache2 to run sudo without providing password. This is usefull when
execute commands on Squid via the dashboard

	sudo touch /etc/sudoers.d/www-data
	sudo vim /etc/sudoers.d/www-data
	www-data ALL=(ALL) NOPASSWD:ALL // add this line in the file

15) Test the application 

	http://192.168.56.120


## How to set up a development environment from an existing box

#### Create a package from a box

	vagrant halt
	vagrant package

Creates a ``package.box``

#### Create a new VM from the new box

1) Add the box to Vagrant in order to be able to build VMs from it 

	vagrant box add <name-of-the-box> <path-to-package.box>
	vagrant box list # to check if the box is listed

2) Create the folder ``dev`` and cd into it

	mkdir dev
	cd dev

3) Clone the ``proxy-build`` repository, and cd into the folder	

		git clone git@github.com:dimosthe/proxy-build.git 
		cd proxy-build    

4) Change the name of the box (as specified in (1)) in ``Vagrantfile`` 

	config.vm.box = "<name-of-the-box>"

5) Create the folder ``shared``

		mkdir shared

This folder will expose some application files and folders to the host filesystem.

6) Create and run the VM

		vagrant up

Once this command finishes, the VM is up and running. Our user is ``vagrant`` and password ``vagrant``. The VM is bound to the local IP ``192.168.56.120``.

7) Now you can use command ``vagrant ssh`` to SSH into the VM, without providing any password.

8) There is an issue with the synced folder. The content is lost after deploying the VM so we need to clone the ``Squid-dashboard`` application again

	sudo -u proxyvnf -s 
	# [Generate SSH keys](https://help.github.com/articles/generating-ssh-keys/) and add the public key to your git profile
	cd /home/proxyvnf/dashboard
	git clone git@github.com:dimosthe/Squid-dashboard.git
	cd Squid-dashboard
	composer install
	
9) Create a new database, a new user and install the yii2-user plugin
	
	mysql -u root -p
	create database dashboarddb
	create user 'dashboarduser'@'localhost' identified by '12345678';
	grant all privileges on dashboarddb.* to dashboarduser@localhost;
	vim config/db.php // edit the file accordingly
	sudo -u proxyvnf -s
	php yii migrate/up --migrationPath=@vendor/dektrium/yii2-user/migrations // in 	order to build the tables for the yii2-user module
	./yii migrate/up // in order to build any extra migrations
	./yii createusers/create // from the root of the application. It creates a default user with username:admin, pass:administrator

10) Test the application 

	http://192.168.56.120

## Deploy Squid 3.5

We build Squid 3.5.5 from source code

#### Requirements

	* g++
	* make
	* autoconf
	* apache2-utils

#### Deploy Squid 3.5.5

1) Download source code, extract and cd into it

2) Run 

	./configure --prefix=/usr --localstatedir=/var
	--libexecdir=${prefix}/lib/squid --srcdir=. --datadir=${prefix}/share/squid
	--sysconfdir=/etc/squid --with-default-user=proxy --with-logdir=/var/log/squid
	--with-pidfile=/var/run/squid.pid --enable-delay-pools
	--enable-auth-basic=DB,NCSA --enable-cache-digests --disable-arch-native

	make
	make install

3) [Build](https://gist.github.com/e7d/1f784339df82c57a43bf#build-service-runtime) ``service squid start``. 

4) Run

	service squid start

5) Add ``basic_db_auth`` plugin under /usr/lib/squid3 directory

## Deploy SquidGuard

1) Install squidguard

	sudo apt-get install squidguard

2) The problem is that squid3 is also installed when running the above command and starts on start-up. In order to disable the service on start-up:

	sudo update-rc.d -f  squid3 remove
	sudo reboot

3) Initializing the blacklists

	sudo squidGuard -C all # convert them from the textfiles to db files. Note that only domains that are defined in the configuration file will be converted
	sudo chown -R proxy:proxy /etc/squidguard/blacklists/* # ensures that squid is able to access the blacklists

4) Configuring Squid

	redirect_program /usr/bin/squidGuard -c /etc/squidguard/squidGuard.conf # at the beginning of squid.conf

## How to migrate a Virtualbox machine to VMware Esxi

[link](https://felixcentmerino.wordpress.com/2014/10/15/migrate-virtual-machine-from-oracle-virtualbox-to-esxi-5-5/)
