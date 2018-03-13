# -*- mode: ruby -*-
# vi: set ft=ruby :

hostname        =  "ansible.dev"

server_ip             = "192.168.22.99"
server_cpus           = "1"   # Cores
server_memory         = "384" # MB
server_swap           = "768" # Options: false | int (MB) - Guideline: Between one or two times the server_memory

server_timezone  = "UTC"

VAGRANTFILE_API_VERSION = "2"

required_plugins = %w( vagrant-hostmanager vagrant-auto_network )

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.ssh.insert_key = false
  config.vm.hostname =  hostname
  config.ssh.forward_agent = true
  
  if Vagrant::Util::Platform.windows?
    config.vm.synced_folder ".", "/vagrant",
    id: "core",
    :nfs => true,
    :mount_options => ["dmode=777", "fmode=777"]
  else
    config.vm.synced_folder ".", "/vagrant",
    id: "core",
    :nfs => true,
    :mount_options => ['nolock,vers=3,udp,noatime,actimeo=2,fsc']
  end


  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = false
  end

  config.vm.provider :virtualbox do |v|
    v.name = hostname 
    v.memory = server_memory
    v.cpus = server_cpus
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
    v.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]
  end

  if Vagrant.has_plugin?("vagrant-auto_network")
    config.vm.network :private_network, :ip => "0.0.0.0", :auto_network => true
  else
    config.vm.network :private_network, ip: server_ip
    config.vm.network :forwarded_port, guest: 80, host: 8181
  end
  
  config.vm.provision "shell", path: "provisioning/scripts/add_repos.sh", privileged: true

  config.vm.provision "shell", path: "provisioning/scripts/base.sh", args: [server_swap, server_timezone]

  config.vm.provision "shell", path: "provisioning/scripts/ansible.sh"

  config.vm.provision "shell", path: "provisioning/scripts/clean_up.sh"

  config.vm.provision "ansible_local" do |ansible|
    ansible.install = false
    ansible.provisioning_path = "/vagrant/provisioning"
    ansible.playbook = "playbooks/nginx.yml"
    ansible.inventory_path = "/vagrant/provisioning/inventory"
    ansible.become = true
    ansible.limit = "all"
  end
  
end