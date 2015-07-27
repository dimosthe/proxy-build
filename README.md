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

