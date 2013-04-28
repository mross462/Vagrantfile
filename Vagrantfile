# -*- mode: ruby -*-
# vi: set ft=ruby :
#  _    __                             __  _____ __
# | |  / /___ _____ __________ _____  / /_/ __(_) /__
# | | / / __ `/ __ `/ ___/ __ `/ __ \/ __/ /_/ / / _ \
# | |/ / /_/ / /_/ / /  / /_/ / / / / /_/ __/ / /  __/
# |___/\__,_/\__, /_/   \__,_/_/ /_/\__/_/ /_/_/\___/
#           /____/

# This is the Vagrantfile that I use to create test enviornments prior to
# deployment on actual hardware. I use ruby hashes to define:
#
#   * Which box the Vagrant configuration will use, if the box isn't present
#     download it from the defined url.
#   * The FQDN for the box
#   * The IP defined for the virtual network of the box
#   * The bridged interface that will be exposed to the network enabling access
#     to the node by others.
#   * Provisioning information that is run after a vagrant up

#Define each configuration in a ruby hash
servers = { "base"          => {"box"   => "base",
                                "box_url"   => "http://files.vagrantup.com/precise64.box"},

            "base_bridged"  => {"box"   => "base",
                                "fqdn"  => "bridged.cloud",
                                "ip"    => "123.123.123.20",
                                "bridged" => "wifi"},

            "base_bridged2"  => {"box"   => "base",
                                "fqdn"  => "bridged2.cloud",
                                "ip"    => "123.123.123.21",
                                "bridged" => "wifi"},

            "geo"           => {"box"       => "mezeo",
                                "box_url"   => "http://tiny.cc/vagrant_box",
                                "fqdn"      => "geo.cloud",
                                "ip"        => "123.123.123.50",
                                "shell_path" => "geo/geo.sh"},

            "mcsp1"         => {"box"       => "mezeo",
                                "box_url"   => "http://tiny.cc/vagrant_box",
                                "fqdn"      => "mcsp1.cloud",
                                "ip"        => "123.123.123.101",
                                "shell_path" => "mcsp1/mcsp1.sh",
                                "bridged"   =>  "wifi",
                                "memory"    => 3072
                               },

            "mcsp2"         => {"box"       => "mezeo",
                                "box_url"   => "http://tiny.cc/vagrant_box",
                                "fqdn"      => "mcsp2.cloud",
                                "ip"        => "123.123.123.102",
                                "shell_path" => "mcsp2/mcsp2.sh",
                                "bridged"   => "wifi",
                                "memory"     => 3072
                               },

            "mcsp3"         => {"box"       => "mezeo",
                                "box_url"   => "http://tiny.cc/vagrant_box",
                                "fqdn"      => "mcsp3.cloud",
                                "ip"        => "123.123.123.103",
                                "shell_path" => "mcsp3/mcsp3.sh",
                                "bridged"   => "wifi",
                                "memory"     => 3072
                                },

            "puppetserver"  => {"fqdn"      => "puppet-server.cloud",
                                "ip"        => "123.123.123.104",
                                "puppet_node" => "vagrant",
                                "puppet_options" => ["--user", "admin@mezeo.com"],
                                "puppet_server" => "puppet.mezeo.local"},

            "puppet"        => {"fqdn"      => "puppet.cloud",
                                 "ip"        => "123.123.123.105",
                                 "manifest_file" => "manifest.pp",
                                 "manifests_path" => "puppet"},

            "chefserver"    => {"chef_server_url" => "http://mychefserver.com:4000",
                                "fqdn"      => "chef-server.cloud",
                                "ip"        => "123.123.123.106",
                                "puppet_node" => "vagrant",
                                "puppet_options" => ["--user", "admin@mezeo.com", "--password", "password"],
                                "validation_key_path" => "chef_server/key.pem"},

            "chef"          => {"cookbooks_paths" => "chef",
                                "ip"        => "123.123.123.106",
                                "runlist" => [],
                                "fqdn"      => "chef.cloud"},

            "server"      => {"fqdn"      => "services.cloud",
                              "ip"        => "123.123.123.100",
                              "bridged"   => "wifi",
                              "shell_path" => "server/services.sh",
                              "memory"    => "2048"},
}

# Here's a quick breakdown on what the single letter objects mean.
# b - box configuration
# c - imported configuration (value in hash above)
# s - server (key in hash above)
# v - vagrant

Vagrant::Config.run do |v|
#Vagrant.configure("2") do |v|

  # For each configuration from the hash above:
  servers.each do |s, c|

    #Define a vagrant instance and do box configuration
    v.vm.define s do |b|

      # Determine which box to use
      if c.has_key?("box")
          b.vm.box = c["box"]
          b.vm.box_url = c["box_url"]
      else
          b.vm.box = "base"
          b.vm.box_url = "http://files.vagrantup.com/precise64.box"
      end

      # Vagrant 1.0 Network Config for Host Only Networking
      if c['ip']
        b.vm.network :hostonly, c["ip"], :netmask => "255.255.0.0"
      end

      # Determine the platform
      if RUBY_PLATFORM.include? 'linux'
        if c['bridged'] == 'wire'
          b.vm.network :bridged, :bridge => "eth0"
        elseif c['bridged'] == 'wifi'
          b.vm.network :bridged, :bridge => "eth1"
        end
      elseif RUBY_PLATFORM.include? 'darwin'
        if c['bridged'] == 'wire'
          b.vm.network :bridged, :bridge => "en0: Ethernet"
        elseif c['bridged'] == 'wifi'
          b.vm.network :bridged, :bridge => "en1: Wi-Fi (AirPort)"
        end
      end

      #Customize memory on the box, default to 1gb
      if c['memory']
        b.vm.customize ["modifyvm", :id, "--memory", c['memory']]
      else
        b.vm.customize ["modifyvm", :id, "--memory", 1024]
      end

      # Configure provider specific information
      #b.vm.provider :virtualbox do |vb|
      #    vb.customize ["modifyvm", :id, 
      #                  "--cpuexecutioncap", "40", 
      #                  "--cpus", "1",
      #                  "--memory", "1200"]

          #Add a host only adapter if it is specified
          #if c['ip']
          #    vb.network_adapter 2, :hostonly, c["ip"]
          #end
          #Add a bridged adapter if it is specified
          #if c['bridged'] 
          #    vb.network_adapter 3, :bridged => "en0: Wi-Fi (AirPort)"
          #end
      #end

      #Update
      b.vm.provision :shell, :inline => "apt-get update"

      #Install The Editor
      b.vm.provision :shell, :inline => "apt-get -y install vim"

      #Install SCMs
      b.vm.provision :shell, :inline => "apt-get -y install git subversion mercurial meld"

      #Install Tools
      b.vm.provision :shell, :inline => "apt-get -y install wget curl"

      # Set the machine hostname based on the server name
      b.vm.provision :shell, :inline => "hostname " + s

      # Modify /etc/hosts all machines in the current Vagrant configuration
      # should know about all the other possible machines they should be
      # dealing with
      b.vm.provision :shell, :inline => "echo >> /etc/hosts"
      servers.each do |s, c|
          if s != 'base'
              hoststring = "echo '" + c["ip"] + " " + s + " " + c['fqdn'] + "' >> /etc/hosts"
              b.vm.provision :shell, :inline => hoststring
          end
      end

      # I like my setup the way I like it, you should too.
      b.vm.provision :shell, :path => "setup.sh"

      # Provision with puppet_server
      if c.has_key?("puppet_server") #and
          #c.has_key?("puppet_node_name")
          b.vm.provision :puppet_server do |ps|
              ps.puppet_server = c["puppet_server"]
              ps.puppet_node = c["puppet_node"]
              ps.options = c["puppet_options"]
          end
      end

      # Provision with puppet
      if c.has_key?("manifests_path") and 
          c.has_key?("manifest_file")
          b.vm.provision :puppet do |p|
              p.manifests_path = c["manifests_path"]
              p.manifest_file = c["manifests_file"]
          end
      end

      # Provision with chef_server
      if c.has_key?("chef_server_url") and
          c.has_key?("validation_key_path")
          b.vm.provision :chef_client do |c| 
              c.chef_server_url = "http://mychefserver.com:4000"
              c.validation_key_path = "chef_server/key.pem"
          end
      end

      # Provision with chef solo
      #if c.has_key?("cookbooks_path") or
      #   c.has_key?("runlist")
      #    b.vm.provision :chef_solo do |c| 
      #        c.cookbooks_path = "chef"
      #    end
      #end

      #Provision with shell
      if c.has_key?("shell_path")
          b.vm.provision :shell, :path => c["shell_path"]
      end
    end
  end
end
