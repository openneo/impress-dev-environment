# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  # Randomly generated 192.168.X.X IP address, to avoid conflicts :)
  config.vm.network "private_network", ip: "192.168.171.5"

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.aliases = %w(impress.dev.openneo.net)

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = {
      "web" => ["default"]
    }
    ansible.verbose = 'v'
  end
end
