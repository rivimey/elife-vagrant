# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Variation configuration: starts with ELIFE_ to indicate these items
# are for this setup.
ELIFE_CONFIGURE_OPENVPN = true         # set to false to disable OpenVPN configuration


# Putting this code here, like this, means it is run whenever the Vagrant command
# is run. At present I don't know of a way to make it more specific to, for example
# 'up' or 'provision' commands.

public_dir = "public"
warnings = 0

# These tests are needed if you are configuring OpenVPN
if ELIFE_CONFIGURE_OPENVPN

  if !File.exist?(public_dir + "/client.crt")
    puts "WARNING: Client VPN certificate client.crt is missing."
    warnings = warnings + 1
  end
  if !File.exist?(public_dir + "/client.key")
    puts "WARNING: Client VPN key client.key is missing."
    warnings = warnings + 1
  end
  if !File.exist?(public_dir + "/ca.crt")
    puts "WARNING: Client Authority certificate ca.crt is missing."
    warnings = warnings + 1
  end
  if !File.exist?(public_dir + "/ta.key")
    puts "WARNING: Client TLS key ta.key is missing."
    warnings = warnings + 1
  end

end

# These tests are needed to run up the Drupal site.

if !File.exist?(public_dir + "/jnl-elife.sql.gz")
  puts "WARNING: Website database dump jnl-elife.sql.gz is missing."
  warnings = warnings + 1
end
if !File.exist?(public_dir + "/settings.php")
  puts "WARNING: Website settings file settings.php is missing."
  warnings = warnings + 1
end

# If there is a problem, let the user have a chance to fix it
if warnings > 0
  print "\nWarnings have been issued: abort this command? [Yes/No] "
  STDOUT.flush
  ans = STDIN.gets.chomp.strip
  abort unless (ans === "no" || ans === "No")
end


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
  config.vm.synced_folder "./public", "/opt/public", id: "vagrant-root", :owner => "www-data", :group => "www-data", :mount_options => [ "dmode=775", "fmode=664" ]

  # this expects the drupal-site-jnl-elife git repository checked out in the same parent folder
  config.vm.synced_folder "../drupal-site-jnl-elife", "/shared/elife_module", id: "vagrant-elife", :owner => "www-data", :group => "www-data", :mount_options => [ "dmode=775", "fmode=664" ]

  # Every Vagrant virtual environment requires a box to build off of.
  # This setting is global to the file; select dummy for AWS otherwise select pre64..
  config.vm.box = "pre64-elife-rb1.9-chef-11"
  config.vm.box_url = "http://cdn.elifesciences.org/vm/pre64-elife-rb1.9-chef-11.box"

# # # # # # # # # # # # # # # # # #

  config.vm.provider :aws do |aws, override|

    override.vm.box = "dummy"
    override.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"

    # Install latest version of Chef
    override.omnibus.chef_version = "11.12.0"

    aws.access_key_id = ENV['AWS_KEY_ID']
    aws.secret_access_key = ENV['AWS_SECRET_KEY']
    aws.keypair_name = ENV['AWS_KEYPAIR_NAME']
    override.ssh.private_key_path = ENV['AWS_KEY_PATH']

    # You cannot pass any parameters to vagrant. The only way is to use
    # environment variables, eg:
    #    MY_VAR='my value' vagrant up
    #    And use ENV['MY_VAR'] in recipe.

    aws.instance_type = "m1.large"
    aws.security_groups = [ "default-vpn" ]

    aws.ami = "ami-2f8f9246"
    aws.region = "us-east-1"

    override.ssh.username = "ubuntu"
    aws.tags = {
      'Name' => 'Elife Vagrant VPN',
     }

  end   # of aws provider

# # # # # # # # # # # # # # # # # #



  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  #
  config.vm.provider :virtualbox do |vb|

    # Boot in headless mode
    vb.gui = false

    #
    # # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "1024"]
    vb.customize ["modifyvm", :id, "--vram", "12"]

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    # new syntax is now
    config.vm.network :private_network, ip: "192.168.33.44"

  end    # of vbox provider

# # # # # # # # # # # # # # # # # #

  config.vm.provision "chef_solo" do |chef|

    # define where things have been collected together by librarian-chef
    chef.cookbooks_path = ["cookbooks"]

    # Set this to :debug if you want more debugging info, else :info or :warn
    chef.log_level = :info

    # This represents our default Drupal development stack.
    chef.add_recipe "elife-drupal-cookbook::drupal_lamp_dev"

    # site and SQL files for our Drupal site
    chef.add_recipe "drupal-site-jnl-elife-cookbook::default"

    # Pulled out so it's obvious: disable content delivery as it won't work for non-live sites
    # apache restart needed before this works and restarts are delayed by default. Change
    # the web_app rule to change this behaviour:
    chef.add_recipe "drupal-site-jnl-elife-cookbook::disable-cdn"

    # we set these attrbutes, and in particular the mysql root password
    # as in chef solo we don't have access to a chef server
    chef.json = {
      "git_root" => "/opt/public",                # directory containing webroot in which drupal repos fetched
      "www_root" => "/opt/public/drupal-webroot", # location of drupal webroot. Not really configurable!
                                                  # because dir is git_root and name is the drupal-webroot git
                                                  # repo folder name.
      "apache" => {
        "listen_ports" => [ "80", "8080" ],         # list of ports to listen on, e.g. [ "80", "8080"]
      },
      "drupal" => {
        "site_name" => "elife.vbox.local",        # a single name by which the server is known
        "site_aliases" => [],                     # used in web_app recipe, alternate names for server
        "shared_folder" => "/vagrant/public",     # in-VM folder used to mount .../elife-vagrant/public
        "drupal_sqlfile" => "jnl-elife.sql",      # Base filename of SQL database dump. gzip compressed as ".gz"
      },
      "mysql" => {
        "server_database" => "jnl_elife",         # the name of the database. Must match settings.php
        "server_root_userid" => "admin",          # database admin userid
        "server_root_password" => "admin",        # database admin password
        "server_repl_password" => "",             # ... not used
        "server_debian_password" => "root",       # ... not used
        "elife_user_password" => "elife",         # ... not used
      },
      "elifejnl" => {
        # revision of highwire module to fetch: 7.x-1.x-stable is prod;  7.x-1.x-dev is dev (not for eLife use)
        "hiwire_rev" => "7.x-1.x-stable",
        "webroot_rev" => "7.x-1.x-stable",
        "setup_vpn_client" => ELIFE_CONFIGURE_OPENVPN
      },
      "apt" => {
        "periodic_update_min_delay" => 720,
      }
    }

  end   # of chef_solo provision

end
