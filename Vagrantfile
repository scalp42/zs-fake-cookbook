# -*- mode: ruby -*-
# vi: set ft=ruby :

RUN_LIST = {
  :zs1 => ['recipe[zs-fake::default]'],
  :aws1 => ['recipe[zs-fake::default]']
}

USER_CONFIG = {
  :chef => {
    :berkshelf => true,
    :omnibus_version => '11.6.0',
    :environment => 'debug'
  },
  :virtualbox => {
    :config => {
      :memory => 512,
      :cpus => 1 },
    :box => 'ubuntu-12.04-audax-chef',
    :url => 'https://s3.amazonaws.com/zs-vagrant/ubuntu-12.04-audax-chef.box'
  },
  :aws => {
    :flavor => 'm1.small',
    :region => 'us-east-1',
    :domain => 'audax.in'
  }
}

Vagrant.configure('2') do |config|
  config.vm.provider :virtualbox do |vb|
      vb.customize OPTS
  end

  config.ssh.forward_agent = true
  config.berkshelf.enabled = USER_CONFIG[:chef][:berkshelf]

  config.vm.define :zs1 do |zs|
    zs.vm.hostname = "zs-fake-berkshelf-zs1-#{ENV['USER']}"

    zs.vm.box = USER_CONFIG[:virtualbox][:box]
    zs.vm.box_url = USER_CONFIG[:virtualbox][:url]

    zs.vm.network :private_network, ip: '33.33.33.11'
    # zs.vm.network :public_network
    # zs.vm.network :forwarded_port, guest: 8081, host: 8081

    zs.ssh.max_tries = 40
    zs.ssh.timeout   = 120

    zs.omnibus.chef_version = USER_CONFIG[:chef][:omnibus_version]

    # `libnss-mdns` is already installed in `ubuntu-12.04-audax-chef` box
    # zs.vm.provision :shell, :inline => 'apt-get --no-install-recommends -y install libnss-mdns'

    zs.vm.provision :chef_client do |chef|
      chef.chef_server_url = ENV['KNIFE_CHEF_SERVER']
      chef.validation_client_name = ENV['KNIFE_VALIDATION_CLIENT_NAME']
      chef.validation_key_path = ENV['KNIFE_VALIDATION_CLIENT_KEY']
      chef.log_level = 'info'
      chef.environment = USER_CONFIG[:chef][:environment]

      chef.run_list = RUN_LIST[:zs1]
    end
  end

  config.vm.define :aws1 do |zs|
    zs.vm.provider 'aws' do |aws, override|
      aws.tags = {'Name' => "zs-fake-berkshelf-aws1-#{ENV['USER']}"}
      aws.access_key_id = "#{ENV['AWS_ACCESS_KEY_ID']}"
      aws.secret_access_key = "#{ENV['AWS_SECRET_ACCESS_KEY']}"
      override.ssh.username = 'ubuntu'
      override.ssh.private_key_path = "/Users/#{ENV['USER']}/.ssh/id_rsa"
      aws.region = USER_CONFIG[:aws][:region]
      aws.region_config USER_CONFIG[:aws][:region] do |region|
        region.ami = 'ami-a29943cb'
        region.keypair_name = "#{ENV['AWS_SSH_KEY_ID']}"
        region.security_groups = ['antho-cuisine', 'chef-clients']
        region.instance_type = USER_CONFIG[:aws][:flavor]
      end
      aws.user_data = "#!/bin/bash\nhostname zs-fake-berkshelf-aws1-#{ENV['USER']}.#{USER_CONFIG[:chef][:environment].gsub('_', '')}.#{USER_CONFIG[:aws][:domain]}\necho 'zs-fake-berkshelf-aws1-#{ENV['USER']}.#{USER_CONFIG[:chef][:environment].gsub('_', '')}.#{USER_CONFIG[:aws][:domain]}' > /etc/hostname\necho -e \"127.0.0.1 `hostname` `hostname -s`\" | sudo tee -a /etc/hosts"
    end

    zs.vm.hostname = "zs-fake-berkshelf-aws1-#{ENV['USER']}.#{USER_CONFIG[:chef][:environment].gsub('_', '')}.#{USER_CONFIG[:aws][:domain]}"

    zs.vm.box = 'ubuntu_aws'
    zs.vm.box_url = 'https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box'

    zs.omnibus.chef_version = USER_CONFIG[:chef][:omnibus_version]

    zs.vm.provision :chef_client do |chef|
      chef.chef_server_url = ENV['KNIFE_CHEF_SERVER']
      chef.validation_client_name = ENV['KNIFE_VALIDATION_CLIENT_NAME']
      chef.validation_key_path = ENV['KNIFE_VALIDATION_CLIENT_KEY']
      chef.log_level = 'info'
      chef.environment = USER_CONFIG[:chef][:environment]

      chef.run_list = RUN_LIST[:aws1]
    end
  end
end

VM_CONFIG = {
  :ioapic => 'on',
  :pae => 'on',
  :chipset => 'ich9',
  :hwvirtex => 'on',
  :nestedpaging => 'on',
  :audio => 'none',
  :usb => 'off'
}

OPTS = ['modifyvm', :id]

VM_CONFIG.merge!(USER_CONFIG[:virtualbox][:config])
VM_CONFIG.each do |k, v|
  OPTS << "--#{k}"
  OPTS << "#{v}"
end
