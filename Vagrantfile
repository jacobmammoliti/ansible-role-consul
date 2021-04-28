Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"

  config.vm.define "consulserver" do |consulserver|
    consulserver.vm.box = "bento/ubuntu-18.04"
    consulserver.vm.hostname = "consul-server"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.groups = {
      "consul" => ["consulclient"]
    }
  end
end