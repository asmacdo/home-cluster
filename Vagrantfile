ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|
  # if Vagrant.has_plugin?("vagrant-cachier")
  #   config.cache.scope = :box
  #   config.cache.synced_folder_opts = {
  #     type: 'rsync',
  #   }
  # end
  config.vm.define 'pxe' do |pxe|
    pxe.vm.box = "fedora/34-cloud-base"
    pxe.vm.provision :file, :source => "~/.ssh/id_rsa.pub", :destination => "/tmp/key"
    pxe.vm.provision :file, :source => "./pxelinux.cfg", :destination => "/tmp/pxe"
    pxe.vm.provision :shell, :inline => <<-EOF
      cat /tmp/key >> /home/vagrant/.ssh/authorized_keys
      mkdir -p /root/.ssh
      cp /home/vagrant/.ssh/authorized_keys /root/.ssh/authorized_keys
    EOF
    pxe.vm.provision :shell, :inline => <<-EOF
      dnf install -y tftp-server podman
      systemctl enable --now tftp
      cp /tmp/pxe /var/lib/tftpboot/pxelinux.cfg
      podman run --privileged --pull=always --rm -v /var/lib/tftpboot/:/data -w /data \
      quay.io/coreos/coreos-installer:release download -f pxe
      dnf install syslinux -y
      cp /usr/share/syslinux/{pxelinux.0,menu.c32,vesamenu.c32,ldlinux.c32,libcom32.c32,libutil.c32} /var/lib/tftpboot/
      # dnf install shim-x64 grub2-efi-x64 --installroot=/tmp/fedora
      # mkdir -p /var/lib/tftpboot/uefi
      # cp /tmp/fedora/boot/efi/EFI/fedora/{shimx64.efi,grubx64.efi} /var/lib/tftpboot/uefi/
    EOF
    pxe.vm.network :private_network,
      :mac => "52:11:22:33:44:41",
      :ip => '192.168.17.11',
      :libvirt__network_name => "home-cluster",
      :libvirt__dhcp_enabled => true,
      :libvirt__netmask => "255.255.255.0",
      :libvirt__dhcp_bootp_file => "pxelinux.0",
      :libvirt__dhcp_bootp_server => "192.168.17.11"
  end
  config.vm.define "somethingelse" do |node|
    # node.hostmanager.manage_guest = false
    node.vm.hostname = "somethingelse"
    node.vm.provider :libvirt do |domain|
      domain.storage :file, :size => '8G', :type => 'qcow2'
      domain.mgmt_attach = 'false'
      domain.management_network_name = 'home-cluster'
      domain.management_network_address = "192.168.17.0/24"
      domain.management_network_mode = "nat"
      domain.boot 'network'
      domain.boot 'hd'
    end
  end
  # config.vm.define 'node' do |node|
  #   pxe.vm.provision :shell, :inline => <<-EOF
  #   EOF
  # end

  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
    libvirt.memory = 1000
    libvirt.cpus = `grep -c ^processor /proc/cpuinfo`.to_i / 2
    # libvirt.qemu_use_session = false
  end
end
