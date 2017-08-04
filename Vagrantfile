# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
require 'vagrant/util/deep_merge'


##########################
##
## Defined default parameters
##

default_params = {
  'type' => 'virtualbox',
  'os' => {
    'cpu' => '1',
    'memory' => '1024',
    'hostname' => '',
  },
  'networks' => {
    'nat_forwarded_port' => [
    ],
    'private' => [
    ],
    'public' => [
    ],
  },
  'vm' => {
    'box' => 'centos/7',
    'box_url' => '',
    'name' => '',
    'gui' => false,
    'box_check_update' => false,
    'sync_folder' => {
      'enable' => false,
      'host' => '.',
      'guest' => '/vagrant',
      'owner' => 'root',
      'group' => 'root'
    }
  },
  'ssh' => {
    'public_key' => '',
  }
}

default_nat_forwarded_port_param = {
  'id' => '',
  'guest' => -1,
  'guest_ip' => nil,
  'host' => -1,
  'host_ip' => nil,
  'protocol' => nil,
  'auto_correct' => false,
}

default_private_network_param = {
  'dhcp' => true,
  'ip_addr' => '',
  'netmask' => '255.255.255.0',
  'auto_config' => true,
  'name' => '',
}

default_public_network_param = {
  'dhcp' => true,
  'dhcp_assigned_default_route' => false,
  'ip_addr' => '',
  'netmask' => '255.255.255.0',
  'bridge' => [],
  'auto_config' => true,
}


##
# Load parameters and merge parameters
#
# @return merged parameters
#
def loadAndMergeParams(params)
  begin
    yml = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))
    unless yml
      yml = { }
    end
  rescue Errno::ENOENT => ex
    yml = { }
  end

  conf = Vagrant::Util::DeepMerge.deep_merge(params, yml)

  return conf
end


Vagrant.configure("2") do |config|
  conf = loadAndMergeParams(default_params)

  # Set box
  config.vm.box = conf['vm']['box']
  if not conf['vm']['box_url'].empty?
    config.vm.box_url = conf['vm']['box_url']
  end

  # Set check box updated
  config.vm.box_check_update = conf['vm']['box_check_update']

  # Set vagrant cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  # Setting Guest OS
  if conf['type'].empty? && conf['type'] == 'virtualbox'
    config.vm.provider "virtualbox" do |v|
      # Guest VM name of VirtualBox
      if not conf['vm']['name'].empty?
        v.name = conf['vm']['name']
      end

      # View VirtualBox GUI
      v.gui = conf['vm']['gui']

      # The number of CPU of guest VM
      v.cpus = conf['os']['cpu']

      # The number of Memory (MB) of guest VM
      v.memory = conf['os']['memory']

      # Set architecture of guest VM
      v.customize ["modifyvm", :id, "--ostype", "RedHat_64"]
      v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end

  if Vagrant.has_plugin?("vagrant-aws")
    if conf['type'] == 'aws'
#      config.vm.provider "aws" do |aws, override|
#        aws.access_key_id = ''
#        aws.secret_access_key = ''
#        aws.keypair_name = ''
#        aws.ami = ''
#        aws.instance_type = ''
#        aws.region = ''
#        aws.availability_zone = ''
#        aws.security_group = []
#        aws.tags = {
#          'Name' = > '',
#          'Description' = > ''
#        }
#        aws.associate_public_ip = true
#        # EBS
#        aws.block_device_mapping = [
#          {
#            'DeviceName' => '/dev/sda1',
#            'VirtualName' => 'ボリューム名',
#            'Ebs.VolumeSize' => 8,
#            'Ebs.VolumeTYpe' => 'standard',
#            #'Ebs.VolumeTYpe' => 'io1',
#            #'Ebs.Iops' => 1000
#          }
#        ]
#        # VPC
#        aws.subnet_id = ''
#        aws.private_ip_address = ''
#        aws.elastic_ip = false
#        # Override SSH
#        override.ssh.username = 'ec2-user'
#        override.ssh.private_key_path = ''
    end
  end

  if Vagrant.has_plugin?("vagrant-vspere")
    if conf['type'] == 'vshere'
#      config.vm.provider "vsphere" do |vs|
#        vs.host = ''
#        vs.data_center_name = ''
#        vs.compute_resource_name = ''
#        vs.resource_pool_name = ''
#        vs.data_store_name = ''
#        vs.template_name = ''
#        vs.name = ''
#        vs.password = ''
#        vs.insecure = true
#        vs.template_name = ''
#        vs.customization_spec_name = ''
    end
  end

  # Set vm hostname
  if not conf['os']['hostname'].empty?
    config.vm.hostname = conf['os']['hostname']
  end


  ##########################
  ##
  ##  Setting shared folder
  ##

  # Set shared folder
  config.vm.synced_folder ".", "/vagrant", disabled: true
  if conf['vm']['sync_folder'].include?('enable') && conf['vm']['sync_folder']['enable']
    config.vm.synced_folder conf['vm']['sync_folder']['host'], conf['vm']['sync_folder']['guest'],
        type: conf['virtualbox'],
        create: true,
        owner: conf['vm']['sync_folder']['owner'],
        group: conf['vm']['sync_folder']['group']
  end


  ##########################
  ##
  ##  Setting network
  ##

  # Setting nat network
  if conf['networks'].include?('nat_forwarded_port')
    conf['networks']['nat_forwarded_port'].each do |net|
      param = Vagrant::Util::DeepMerge.deep_merge(default_nat_forwarded_port_param, net)

      config.vm.network "forwarded_port",
          id: param['id'],
          guest: param['guest'],
          guest_ip: param['guest_ip'],
          host: param['host'],
          host_ip: param['host_ip'],
          protocol: param['protocol'],
          auto_correct: param['auto_correct']
    end
  end

  # Setting private network
  if conf['networks'].include?('private')
    conf['networks']['private'].each do |net|
      param = Vagrant::Util::DeepMerge.deep_merge(default_private_network_param, net)

      if net['dhcp']
        config.vm.network "private_network",
            type: 'dhcp'

      elsif net['name'].nil? || net['name'].empty?
        config.vm.network "private_network",
            ip: param['ip_addr'],
            netmask: param['netmask'],
            auto_config: param['auto_config']

      else
        config.vm.network "private_network",
            ip: param['ip_addr'],
            netmask: param['netmask'],
            virtualbox__intnet: param['name'],
            auto_config: param['auto_config']

      end
    end
  end

  # Setting public network
  if conf['networks'].include?('public')
    conf['networks']['public'].each do |net|
      param = Vagrant::Util::DeepMerge.deep_merge(default_public_network_param, net)

      # Setting bridge-adaptor
      if net['dhcp']
        config.vm.network "public_network",
            use_dhcp_assigned_default_route: param['dhcp_assigned_default_route']

      else
        config.vm.network "public_network",
            ip: param['ip_addr'],
            netmask: param['netmask'],
            bridge: param['bridge'],
            auto_config: param['auto_config']
      end
    end
  end

  # ネットワークを追加した際に、追加したネットワークを有効にする
  config.vm.provision "shell", inline: "sudo systemctl restart network", run: always


  ##########################
  ##
  ##  Copy ssh public key
  ##

  if conf['ssh']['public_key']
    # Copy public key
    config.vm.provision "file",
      source: File.join(File.dirname(__FILE__), conf['ssh']['public_key']),
      destination: "~/.ssh/host.id_rsa.pub"

    # Append authorized_keys
    config.vm.provision "shell" do |s|
      s.privileged = false
      s.inline = <<-SHELL
        key=`cat ~/.ssh/host.id_rsa.pub`
        if [ `cat ~/.ssh/authorized_keys | grep "$key" | wc -l` = "0" ]; then
          cat ~/.ssh/host.id_rsa.pub >> ~/.ssh/authorized_keys
        fi
      SHELL
    end
  end

  config.ssh.forward_agent = true
  config.ssh.insert_key = false
  config.ssh.pty = false
end
