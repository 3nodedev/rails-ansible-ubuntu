# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.ssh.forward_agent = true

  config.ssh.shell = %{bash -c 'BASH_ENV=/etc/profile exec bash'}

  # Required for NFS to work, pick any local IP
  config.vm.network :private_network, ip: '172.28.128.5', hostsupdater: 'skip'

  config.vm.hostname = "test-rails"
  config.vm.synced_folder "data", "/data", create: true, type: "nfs"
  config.bindfs.bind_folder "/data", "data"

  host = RbConfig::CONFIG['host_os']
  if host =~ /darwin/
    cpus = `sysctl -n hw.ncpu`.to_i
    memory = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
  elsif host =~ /linux/
    cpus = `nproc`.to_i
    memory = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
  else
    cpus = 2
    memory = 4096
  end

  config.vm.provider 'virtualbox' do |vb|
    vb.customize ['modifyvm', :id, '--memory', memory]
    vb.customize ['modifyvm', :id, '--cpus', cpus]

    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']

    vb.name = config.vm.hostname
  end

  # VMware Workstation/Fusion settings
  ['vmware_fusion', 'vmware_workstation'].each do |provider|
    config.vm.provider provider do |vmw, override|
      override.vm.box = 'puppetlabs/ubuntu-14.04-64-nocm'

      vmw.vmx['memsize'] = memory
      vmw.vmx['numvcpus'] = cpus

      vmw.name = config.vm.hostname
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.sudo = true
    ansible.verbose = "true"
  end
end
