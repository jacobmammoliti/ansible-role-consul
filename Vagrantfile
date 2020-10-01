Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.define "consul-server"
  config.vm.hostname = "consul-server"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yaml"
    ansible.groups = {
      "servers" => ["consul-server"],
    }
  end
end

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.define "consul-client"
  config.vm.hostname = "consul-client"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yaml"
    ansible.groups = {
      "clients" => ["consul-client"],
    }
  end
end

