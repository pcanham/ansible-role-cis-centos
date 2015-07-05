# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
echo "gem: --no-rdoc --no-ri" >> ~/.gemrc
yum localinstall -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y python-setuptools ansible
SCRIPT

unless Vagrant.has_plugin?("vagrant-ansible-local")
  raise 'Following vagrant plugin is missing vagrant-ansible-local'
end

unless Vagrant.has_plugin?("vagrant-cachier")
  raise 'Following vagrant plugin is missing vagrant-cachier'
end

Vagrant.configure("2") do |config|

  config.vm.box = "puppetlabs/centos-7.0-64-puppet"
  config.vm.box_check_update = true

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.provider :virtualbox do |vb, override|
    vb.gui = true
    vb.customize [
      "modifyvm", :id,
      "--memory", "512",
      "--cpus", "4",
      "--natdnspassdomain1", "off",
      ]
  end

  config.vm.provider :vmware_fusion do |v, override|
      v.vmx["memsize"] = 1024
      v.vmx["numvcpus"] = 4
  end

## NOTE: "_" in box names are converted into "-" so that DNS works.
boxes = [
  { :name => :base_image, :ip => '10.0.0.20', :memory => 1024, :ansibleplaybook => 'testing.yml' }
]

boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:memory] ]
      end
      config.vm.provider :vmware_fusion do |v|
        v.vmx["memsize"] =  opts[:memory]
      end
      config.vm.network   :private_network, ip: opts[:ip]
      config.vm.hostname  = "%s.sandbox.internal" % opts[:name].to_s.gsub('_', '-')
      config.vm.provision :shell, :inline => $script
      config.vm.provision :ansibleLocal, :playbook => opts[:ansibleplaybook]
    end
  end
end
