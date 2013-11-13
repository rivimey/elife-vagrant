# About

This repo contains the provisioning information for setting up a dev environment for building a local copy of [drupal-site-jnl-elife](https://github.com/elifesciences/drupal-site-jnl-elife).

It is being built on top of the [drupal - vagrant project](https://drupal.org/project/vagrant), with the configuration moved to v2 of the vagrant API, and using librarian and chef for managing cookbooks.

# Preqrequisites:

Git on local host
Git user created and with access to the elifesciences repository
VirtualBox v4.2 installed (needed for shared folders)
Vagrant installed on local host
Tunnelblick installed on the host

# Setup

To setup your system to work with Vagrant and Chef please follow the guide provided at [elife-template-env](https://github.com/elifesciences/elife-template-env).

The first time that you run vagrant it will attempt to detect whether you have the required base box on your machine. If you haven't then it will download this for you. If you wish to run this step manually then you can do so with the following command:

	vagrant box add pre64-elife-rb1.9-chef-11 http://cdn.elifesciences.org/vm/pre64-elife-rb1.9-chef-11.box

Set up Tunnelblick to connect to Highwire using the key provided, and then connect.

# Quickstart

Currently, use branch ruth-vagrant-fixes for the elife-vagrant module.

	git clone git@github.com:elifesciences/elife-vagrant.git

	### change elifesciences to highwire in the next line if you have access and want the official repo: ###
	git clone git@github.com:elifesciences/drupal-highwire.git

	git clone git@github.com:elifesciences/drupal-site-jnl-elife.git
	cd elife-vagrant

Now, Locate a copy of the journal database dump and save as a gzipped compressed file to .../elife-vagrant/public/jnl-elife.sql.gz, and locate a copy of the settings.php file (with drupal passwords and database setup) and save in .../elife-vagrant/public/settings.php

	librarian-chef install && vagrant up

	## not necessary, but to check the server out, do:
	vagrant ssh

The site is set up to accept connections using http://elife.vbox.local:8080// so on the host configure /etc/hosts to include/extend a line such as:

	127.0.0.1    localhost  elife.vbox.local


# Restarting

Some changes can be made using 'vagrant provision', but it is not recommended because changes aren't necessarily noticed and redone correctly:

	vagrant provision

To remake, following a change in the config files, do:

	cd elife-vagrant      # in this directory
	rm -rf cookbooks  tmp  Cheffile.lock
	librarian-chef install && vagrant up



# What it does

Cookbooks are installed using librarian. Check the `Cheffile` to see which cookbooks we donwnlad as community cookbooks, and which ones we have made private instances of. (Creating a private instance of a cookbook is an anti-pattern, however for our immediate purposes, this is OK).

Librarian also installs the git repo for the elife specific drupal modules and themes.

This git repo includes the `roles` provided by the drupal vagrant proejct. These have been modified to not install varnish or memcached. This was done out of convinience.

This git repo contains a placeholder `public` directory. This is the directory that is used to serve our drupal site out of.


## elife journal drupal code

- [drupal-site-jnl-elife][eldcode] - the source code for the elife drupal site
- [drupal-site-jnl-elife-env][eldprovision] - Vagrant and Cheffile for building the vm
- [drupal-site-jnl-elife-cookbook][eldcook] - Chef cookbook called by Cheffile for building the dev environment

[eldcode]: https://github.com/elifesciences/drupal-site-jnl-elife
[eldprovision]: https://github.com/elifesciences/drupal-site-jnl-elife-env
[eldcook]: https://github.com/elifesciences/drupal-site-jnl-elife-cookbook

## Issues with the Drupal Install

## TODO

- set the memory limit to 256M automatically instead of requiring this step to be run manually after the virtual machine is set up

- the roles are a little over complicated, these should be simplified

- `cookbooks/drupal/recipes/drupal_apps.rb` has been modified to not check whether the key `#if node["hosts"].has_key("localhost_aliases")` exists. Probably an updated issues with Ruby, should check, and re-implement

- apc has been turned off, as getting it to install via chef was beyond me.

- contribute back fixes to the main vagrant-drupal project, and stop holiding a private version of these cookbooks.
