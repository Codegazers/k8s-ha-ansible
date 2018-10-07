# -*- mode: ruby -*-
# vi: set ft=ruby :
#Vagrant::DEFAULT_SERVER_URL.replace('https://vagrantcloud.com')
# Require YAML module
require 'yaml'

config = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))

base_box=config['environment']['base_box']
base_box_version=config['environment']['base_box_version']
master_ip=config['environment']['masterip']
domain=config['environment']['domain']

engine_version=config['environment']['engine_version']

boxes = config['boxes']

boxes_hostsfile_entries=""

 boxes.each do |box|
   boxes_hostsfile_entries=boxes_hostsfile_entries+box['mgmt_ip'] + ' ' +  box['name'] + ' ' + box['name']+'.'+domain+'\n'
 end

#puts boxes_hostsfile_entries

disable_swap = <<SCRIPT
    swapoff -a 
    sed -i '/swap/{ s|^|#| }' /etc/fstab
SCRIPT

update_hosts = <<SCRIPT
    echo "127.0.0.1 localhost" >/etc/hosts
    echo -e "#{boxes_hostsfile_entries}" |tee -a /etc/hosts
SCRIPT

common_packages_and_config = <<SCRIPT
  echo "useDNS no" >> /etc/ssh/sshd_config
  systemctl restart ssh
  systemctl stop apt-daily.timer
  systemctl disable apt-daily.timer
  sed -i '/Update-Package-Lists/ s/1/0/' /etc/apt/apt.conf.d/10periodic
  while true;do fuser -vki /var/lib/apt/lists/lock || break ;done
  apt-get update -qq && apt-get install -qq ntpdate ntp && timedatectl set-timezone Europe/Madrid
SCRIPT

install_docker_engine = <<SCRIPT
  #curl -sSk $1 | sh
  DEBIAN_FRONTEND=noninteractive apt-get remove -qq docker docker-engine docker.io
  DEBIAN_FRONTEND=noninteractive apt-get update -qq
  DEBIAN_FRONTEND=noninteractive apt-get install -qq \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common \
  bridge-utils
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | DEBIAN_FRONTEND=noninteractive apt-key add -
  add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
  DEBIAN_FRONTEND=noninteractive apt-get -qq update
  DEBIAN_FRONTEND=noninteractive apt-get install -y docker-ce=$1
  usermod -aG docker vagrant >/dev/null
SCRIPT

ansible_enablement = <<SCRIPT
  DEBIAN_FRONTEND=noninteractive apt-get install -qq python
  useradd -m -s /bin/bash provision
  mkdir -p ~provision/.ssh
  cp /vagrant/keys/provision.pub ~provision/.ssh/authorized_keys
  chown -R provision:provision ~provision/.ssh
  chmod -R 700 ~provision/.ssh
  echo "provision ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/provision
  echo "Defaults:provision !requiretty" >>/etc/sudoers.d/provision
  
SCRIPT

Vagrant.configure(2) do |config|
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"       	      
  config.vm.box = base_box
  config.vm.box_version = base_box_version
  config.vm.synced_folder "tmp_deploying_stage/", "/tmp_deploying_stage",create:true
  config.vm.synced_folder "examples/", "/examples",create:true
  boxes.each do |node|
    config.vm.define node['name'] do |config|
      config.vm.hostname = node['name']
      config.vm.provider "virtualbox" do |v|
    	  v.linked_clone = true
        v.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
        v.name = node['name']
        v.customize ["modifyvm", :id, "--memory", node['mem']]
        v.customize ["modifyvm", :id, "--cpus", node['cpu']]
        v.customize ["modifyvm", :id, "--nictype1", "Am79C973"]
        v.customize ["modifyvm", :id, "--nictype2", "Am79C973"]
        v.customize ["modifyvm", :id, "--nictype3", "Am79C973"]
        v.customize ["modifyvm", :id, "--nictype4", "Am79C973"]
        v.customize ["modifyvm", :id, "--nicpromisc1", "allow-all"]
        v.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
        v.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
        v.customize ["modifyvm", :id, "--nicpromisc4", "allow-all"]
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

      end

      config.vm.network "private_network",
      ip: node['mgmt_ip'],:netmask => "255.255.255.0",
      virtualbox__intnet: false,
      hostonlyadapter: ["vboxnet1"]

      config.vm.network "private_network",
      ip: node['mgmt_ip'],:netmask => "255.255.255.0",
      virtualbox__intnet: false,
      hostonlyadapter: ["vboxnet2"]

      config.vm.network "public_network",
      bridge: ["enp4s0","wlp3s0","enp3s0f1","wlp2s0","enp3s0"],
      auto_config: true

      config.vm.provision :shell, :inline => common_packages_and_config

      config.vm.provision :shell, :inline => update_hosts
      
      config.vm.provision :shell, :inline => disable_swap

      config.vm.provision :shell, :inline => ansible_enablement

      
      config.vm.provision "shell", inline: <<-SHELL
        sudo cp -R /examples ~vagrant
        sudo chown -R vagrant:vagrant ~vagrant/examples
      SHELL
 
      ## INSTALLDOCKER --> on script because we can reprovision
      config.vm.provision "shell" do |s|
     		s.name       = "Install Docker Engine version "+engine_version
        s.inline     = install_docker_engine
       	s.args       = engine_version
      end


      
    end
  end

end
