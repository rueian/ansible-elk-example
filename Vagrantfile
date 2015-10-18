VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false

  config.vm.box = "ubuntu/trusty64"

  config.vm.synced_folder '.', '/vagrant'

  config.vm.provider "virtualbox" do |v|
    v.memory = 512
    v.cpus = 4
  end

  (10..12).each do |i|
    config.vm.define "#{i}.elastic" do |node|
      node.vm.hostname = "#{i}.elastic"
      node.vm.network :private_network, ip: "10.10.10.#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 768
      end
    end
  end

  (10..14).each do |i|
    config.vm.define "#{i}.logstash" do |node|
      node.vm.hostname = "#{i}.logstash"
      node.vm.network :private_network, ip: "10.10.20.#{i}"
    end
  end

  (10..10).each do |i|
    config.vm.define "#{i}.postgres" do |node|
      node.vm.hostname = "#{i}.postgres"
      node.vm.network :private_network, ip: "10.10.30.#{i}"
    end
  end

  config.vm.define "10.nginx" do |node|
    node.vm.hostname = "10.nginx"
    node.vm.network :private_network, ip: "10.10.40.10"

    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "main.yml"
      ansible.limit = "all"
      ansible.inventory_path = "inventory.ini"
      ansible.extra_vars = {
        ansible_ssh_user: "vagrant",
        ansible_ssh_pass: "vagrant"
      }
    end
  end
end
