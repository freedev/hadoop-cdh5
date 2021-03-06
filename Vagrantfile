# -*- mode: ruby -*-
# vi: set ft=ruby :

current_dir = File.dirname(__FILE__)

require 'yaml'
require "#{current_dir}/libraries/myvagrantlib.rb"

mylib = MyVagrantLib.new

mylib.check_plugins

provider = mylib.get_provider("#{current_dir}/config/provisioner.yml")

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = '2'

Vagrant.require_version '>= 1.5.0'

# Import Virtualbox configs from YAML file.
vb_conf = YAML.load_file "#{current_dir}/config/provisioners/#{provider}/config.yml"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  config.vm.hostname = 'hadoop-cdh5-berkshelf'

  # Set the version of chef to install using the vagrant-omnibus plugin
  # NOTE: You will need to install the vagrant-omnibus plugin:
  #
  #   $ vagrant plugin install vagrant-omnibus
  #
  if Vagrant.has_plugin? 'vagrant-omnibus'
    config.omnibus.chef_version = 'latest'
  end

  # Every Vagrant virtual environment requires a box to build off of.
  # If this value is a shorthand to a box in Vagrant Cloud then
  # config.vm.box_url doesn't need to be specified.

  if provider == :virtualbox
    puts "* Using virtualbox *"
    config.vm.box = 'chef/ubuntu-14.04'
    config.vm.synced_folder ".", "/vagrant", type: "nfs"

    config.vm.provider :virtualbox do |vb|

      vb.customize ["modifyvm", :id, "--cpus", vb_conf['vb_cpu_count']]
#      vb.customize ["modifyvm", :id, "--cpuexecutioncap", vb_conf['vb_cpu_exec_cap']]
      vb.customize ["modifyvm", :id, "--memory", vb_conf['vb_memory']]
      vb.customize ["modifyvm", :id, "--name", vb_conf['vb_name']]
      vb.customize ["modifyvm", :id, "--vram", vb_conf['vb_vram']]
      vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]

    end  
  end

  if provider == :lxc
    puts "* Using lxc *"
    config.vm.box = 'fgrehm/trusty64-lxc'
  end

  # Assign this VM to a host-only network IP, allowing you to access it
  # via the IP. Host-only networks can talk to the host machine as well as
  # any other machines on the same network, but cannot be accessed (through this
  # network interface) by any external networks.
  config.vm.network :private_network, type: 'dhcp'

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # The path to the Berksfile to use with Vagrant Berkshelf
  config.berkshelf.berksfile_path = "#{current_dir}/Berksfile"

  # Enabling the Berkshelf plugin. To enable this globally, add this configuration
  # option to your ~/.vagrant.d/Vagrantfile file
  config.berkshelf.enabled = true

  # An array of symbols representing groups of cookbook described in the Vagrantfile
  # to exclusively install and copy to Vagrant's shelf.
  # config.berkshelf.only = []

  # An array of symbols representing groups of cookbook described in the Vagrantfile
  # to skip installing and copying to Vagrant's shelf.
  # config.berkshelf.except = []

  config.vm.provision :chef_solo do |chef|

    chef.log_level = :debug

    chef.custom_config_path = "#{current_dir}/Vagrantfile.chef"

    chef.json = {
      mysql: {
        server_root_password: 'rootpass',
        server_debian_password: 'debpass',
        server_repl_password: 'replpass'
      },
      :java => {
        :install_flavor => 'oracle',
        :jdk_version => 8,
        :oracle => {
          :accept_oracle_download_terms => true
        }
      }
    }

    chef.cookbooks_path = "#{current_dir}/cookbooks"
    chef.data_bags_path = "#{current_dir}/data_bags"

    chef.run_list = [
      'recipe[hadoop-cdh5::apt-get-update]',
      'recipe[hadoop-cdh5::common-packages]',
      'recipe[apt::cacher-ng]',
      'recipe[hadoop-cdh5::apt-get-update]',
      'recipe[ssh_known_hosts]',
      'recipe[java::default]',
      'recipe[maven::default]',
      'recipe[hadoop-cdh5::apt-get-update]',
      'recipe[hadoop-cdh5::hadoop]'
    ]
  end
end
