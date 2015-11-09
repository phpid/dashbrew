# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

Vagrant.require_version '>= 1.6.0'

Vagrant.configure(2) do |config|

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "mdkholy/dashbrew"

  # The hostname the machine should have.
  config.vm.hostname = "dashbrew.vm"

  # Default Box IP Address
  #
  # This is the IP address that your host will communicate to the guest through.
  #
  # If you are already on a network using the 192.168.10.x subnet, this should be changed.
  # If you are running more than one VM through VirtualBox, different subnets should be used
  # for those as well. This includes other Vagrant boxes.
  config.vm.network :private_network, ip: "192.168.10.10"

  # VirtualBox-specific configuration.
  config.vm.provider "virtualbox" do |vb|
    host = RbConfig::CONFIG['host_os']

    # Give VM 1/4 system memory & access to all cpu cores on the host
    if host =~ /darwin/
        cpus = `sysctl -n hw.ncpu`.to_i
        # sysctl returns Bytes and we need to convert to MB
        mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif host =~ /linux/
        cpus = `nproc`.to_i
        # meminfo shows KB and we need to convert to MB
        mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
        # 6144
    else # sorry Windows folks, I can't help you
        cpus = 2
        mem = 1024
    end

    vb.customize ["modifyvm", :id, "--memory", mem]
    vb.customize ["modifyvm", :id, "--cpus", cpus]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
    #vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    #vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  # Synced Folders
  #
  # For syncing folders between the host and the guest machines.
  # The first parameter is a path to a directory on the host machine.
  # The second parameter must be an absolute path of where to share the folder within the guest machine.
  # Finally the third parameter is a set of non-required options to configure synced folders.


  if Vagrant.has_plugin?("vagrant-bindfs")
    config.nfs.map_uid = Process.uid
    config.nfs.map_gid = Process.gid
    config.vm.synced_folder "public", "/vagrant-nfs", nfs: true, mount_options: ['rw', 'vers=3', 'tcp', 'fsc'], :linux__nfs_options => ['rw','no_subtree_check','all_squash','async']
    config.bindfs.bind_folder "/vagrant-nfs", "/var/www", :owner => "vagrant", :group => "www-data", :'create-as-user' => true, :perms => "u=rwx:g=rwx:o=rD", :'create-with-perms' => "u=rwx:g=rwx:o=rD"
  else
    # the fsc is for cachedfilesd
    config.vm.synced_folder "public", "/var/www", nfs: true, mount_options: ['rw', 'vers=3', 'tcp', 'fsc'], :linux__nfs_options => ['rw','no_subtree_check','all_squash','async']
  end

  # Run the main shell provisioner.
  config.vm.provision 'shell', run: 'always' do |s|
    s.path        = 'provision/provision.sh'
    s.privileged  = true
  end

  # Check if vagrant-hosts-provisioner plugin is installed
  if Vagrant.has_plugin?("vagrant-hosts-provisioner")
    # Run the hostsupdate provisioner
    config.vm.provision :hostsupdate, run: 'always' do |host|
      # Don't include the machine hostname
      host.hostname = false
      # Update the guest machine
      host.manage_guest = true
      # Update the host machine
      host.manage_host = true
      host.aliases = []
      # Pull the hosts entries from the hosts.json file generated by the main provisioner
      host.files = '/provision/main/etc/hosts.json'
    end
  end
end
