VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false

  config.vm.box = "ubuntu/trusty64"

  config.vm.synced_folder '.', '/vagrant'

  config.vm.provider "virtualbox" do |v|
    v.memory = 512
    v.cpus = 4
  end

  (10..11).each do |i|
    config.vm.define "elastic#{i}" do |node|
      node.vm.hostname = "elastic#{i}"
      node.vm.network :private_network, ip: "10.10.10.#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 768
      end
    end
  end

  (10..10).each do |i|
    config.vm.define "logstash#{i}" do |node|
      node.vm.hostname = "logstash#{i}"
      node.vm.network :private_network, ip: "10.10.20.#{i}"
    end
  end

  (10..11).each do |i|
    config.vm.define "logstash-forwarder#{i}" do |node|
      node.vm.hostname = "logstash-forwarder#{i}"
      node.vm.network :private_network, ip: "10.10.30.#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 256
      end
    end
  end

  config.vm.define "kibana" do |node|
    node.vm.hostname = "kibana"
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
