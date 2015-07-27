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
