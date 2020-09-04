# -*- mode: ruby -*-
# vi: set ft=ruby :

cfg = {
  :vagrant_provider => "vbox",
  :vm_count => 1,
  :vm_prefix => "testing",
  :vm_cpus => 2,
  :vm_memory => 2048,
  :vm_box => "generic/ubuntu1804",
  :ssh_priv => "/home/.ssh/id_rsa",
  :ssh_public => "/home/.ssh/id_rsa.pub",
  :public_network_bridge => "wlp4s0",
  :private_network_ip => "192.168.30.0"
}

boxes = [
  {
    :name => "Kubernetes Master",
    :hostname => "kmaster",
    :memory => 2048,
    :cpus => 2,
  },
  {
    :name => "Kubernetes Node 1",
    :hostname => "knode1",
    :memory => 2048,
    :cpus => 2
  }
]

if cfg[:vm_count] > 0
  boxes = []
  for i in 1..cfg[:vm_count] do
    m = {
      :name => cfg[:vm_prefix] + i.to_s,
      :hostname => cfg[:vm_prefix] + i.to_s,
      :memory => cfg[:vm_memory],
      :cpus => cfg[:cpus],
    }
    boxes << m
  end
end

def network_options(cfg)
  options = {}

  if cfg.has_key?('private_network_ip')
    options[:ip] = cfg['private_network_ip']
    options[:netmask] = cfg['netmask'] ||= '255.255.255.0'
  else
    options[:type] = 'dhcp'
  end

  # if host.has_key?('mac')
  #   options[:mac] = host['mac'].gsub(/[-:]/, '')
  # end
  # if host.has_key?('auto_config')
  #   options[:auto_config] = host['auto_config']
  # end
  # if host.has_key?('intnet') && host['intnet']
  #   options[:virtualbox__intnet] = true
  #end

  options
end

Vagrant.configure(2) do |config|
  config.vm.box = "#{cfg[:vm_box]}"
  config.vm.network "public_network", bridge: "#{cfg[:public_network_bridge]}"

  config.vm.provision "file", source: "#{cfg[:ssh_priv]}", destination: "/home/vagrant/.ssh/id_rsa"
  public_key = File.read("#{cfg[:ssh_public]}")
  config.vm.provision :shell, :inline =>"
          echo 'Copying SSH Keys to the VM'
          mkdir -p /home/vagrant/.ssh
          chmod 700 /home/vagrant/.ssh
          echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
          chmod -R 600 /home/vagrant/.ssh/authorized_keys
          echo 'Host *' >> /home/vagrant/.ssh/config
          echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
          echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
          chmod -R 600 /home/vagrant/.ssh/config
          echo 'Connection information:'
          ip=$(hostname -I | awk '{print $1}')
          echo ip: $ip
          echo hostname: $HOSTNAME
          ", privileged: false

  boxes.each do |machine|
    hostname = cfg[:vm_prefix] + "-" + machine[:hostname]
    config.vm.define "#{hostname}" do |m|
      m.vm.hostname = hostname
      m.vm.network "private_network", network_options(machine)  #ip: "#{cfg[:private_network_ip]}"
      m.vm.provider cfg[:vagrant_provider] do |v|
        v.vmname = cfg[:vm_prefix] + ": " + machine[:name]
        v.memory = machine[:memory]
        v.cpus = machine[:cpus]
        v.linked_clone = true
        v.mac = "525400" + Array.new(6){[*"A".."F", *"0".."9"].sample}.join
      end;
    end
  end
end
