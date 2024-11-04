Vagrant.configure(2) do |config|

  config.vm.define "control-plane-node-1" do |vm|
    vm.vm.provider :libvirt do |domain|
      domain.cpus = 2
      domain.uri = 'qemu+unix:///system'
      domain.memory = 2048
      domain.storage :file, :device => :cdrom, :path => "/home/tamay.mueller/Dokumente/IdeaProjects/k8s/dev-environments/talos/talos/metal-amd64.iso"
      domain.storage :file, :size => '10G', :type => 'raw'
      domain.boot 'hd'
      domain.boot 'cdrom'
    end
    vm.vm.network :private_network,
      :ip => "192.168.100.2",
      :libvirt__guest_ipv6 => "yes",
      :libvirt__ipv6_address => "fd00::2",
      :libvirt__ipv6_prefix => "64"
  end

  config.vm.define "worker-node-1" do |vm|
    vm.vm.provider :libvirt do |domain|
      domain.cpus = 1
      domain.uri = 'qemu+unix:///system'
      domain.memory = 2048
      domain.storage :file, :device => :cdrom, :path => "/home/tamay.mueller/Dokumente/IdeaProjects/k8s/dev-environments/talos/talos/metal-amd64.iso"
      domain.storage :file, :size => '10G', :type => 'raw'
      domain.boot 'hd'
      domain.boot 'cdrom'
    end
    vm.vm.network :private_network,
      :ip => "192.168.100.3",
      :libvirt__guest_ipv6 => "yes",
      :libvirt__ipv6_address => "fd00::3",
      :libvirt__ipv6_prefix => "64"
  end

end
