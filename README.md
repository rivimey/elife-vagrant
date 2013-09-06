


# About

This repo contains the provisioning information for setting up a dev environment for building a local copy of [drupal-site-jnl-elife](https://github.com/elifesciences/drupal-site-jnl-elife).

# Setup

To setup your system to work with Vagrant and Chef please follow the guide provided at [elife-template-env](https://github.com/elifesciences/elife-template-env).


# Quickstart

	$ git clone git@github.com:elifesciences/drupal-site-jnl-elife-env.git
	$ cd drupal-site-jnl-elife-env
	$ librarian-chef install
	$ vagrant up
	$ vagrant ssh


## elife journal drupal code

- [drupal-site-jnl-elife][eldcode] - the source code for the elife drupal site
- [drupal-site-jnl-elife-env][eldprovision] - Vagrant and Cheffile for building the vm
- [drupal-site-jnl-elife-cookbook][eldcook] - Chef cookbook called by Cheffile for building the dev environment

[eldcode]: https://github.com/elifesciences/drupal-site-jnl-elife
[eldprovision]: https://github.com/elifesciences/drupal-site-jnl-elife-env
[eldcook]: https://github.com/elifesciences/drupal-site-jnl-elife-cookbook

## Issues with the Drupal Install

## TODO

- `cookbooks/drupal/recipes/drupal_apps.rb` has been modified to not check whether the key `#if node["hosts"].has_key("localhost_aliases")` exists. Probably an updated issues with Ruby, should check, and re-imliment

- apc has been turned off, as getting it to install via chef was beyond me.