# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # need to add apache to the synced folder via
  config.vm.synced_folder "./public", "/vagrant/public", id: "vagrant-root", :owner => "www-data", :group => "www-data"


  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "pre64-elife-rb1.9-chef-11"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
     vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # new syntax is now
  config.vm.network :private_network, ip: "192.168.33.44"

  config.vm.provision "chef_solo" do |chef|

    # This role represents our default Drupal development stack.

    #chef.add_role "drupal_lamp_varnish_dev"

    # Install an example D7 install at drupal.vbox.local.

    # the receipe that we are basing this build off of
    # installs the following: 
    # - varnish 
    # - memcaced

    # as this is for a purely dev install, we are going to ignore these for now. 


    chef.cookbooks_path = ["cookbooks"]
    chef.roles_path = ["roles"]

    # this installs most of the infrastrucutre required to support a drupal instance

    chef.add_recipe 'apt' # add this so we have updated packages available
    chef.add_recipe "git"
    chef.add_recipe "drupal-site-jnl-elife-cookbook"
    chef.add_role("drupal_lamp_varnish_dev")

    # chef.add_role("apache2_mod_php")
    # chef.add_role("apache2_backend")
    # chef.add_role("mysql_server")
    # chef.add_role("drupal")
    # chef.add_role("drupal_dev")
    # chef.add_recipe "drupal::drupal_apps"

    chef.add_recipe 'drupal::example'

    # we set these attrbutes, and in particular the mysql root passwors
    # as in chef solo we don't have access to a chef server
    chef.json = {
            "www_root" => '/vagrant/public',
            "hosts" => {
            "localhost_aliases" => ["drupal.vbox.local", "elife.vbox.local"]#, "drupal.vbox.local"]
            },
            "mysql" => {
                "server_root_password" => "root",
                "server_repl_password" => "root",
                "server_debian_password" => "root",
                "elife_user_password" => "elife"                
                }
        }   


    # chef.add_recipe 'apt'
    # chef.add_recipe 'apache2'
    # chef.add_recipe 'build-essential'
    # chef.add_recipe 'memcached'
    # chef.add_recipe 'mysql'
    # chef.add_recipe 'openssl'
    # chef.add_recipe 'php'
    # chef.add_recipe 'varnish'
    # chef.add_recipe 'drush'
    # chef.add_recipe 'drush_make'
    # chef.add_recipe 'drupal::example'
    # #chef.add_recipe 'hosts'    
    # chef.add_recipe 'phpmyadmin'    
    # chef.add_recipe 'webgrind'    
    # chef.add_recipe 'xhprof'    
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080


  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network



end
