# Set default terminal
ENV["TERM"]="linux"

Vagrant.configure("2") do |config|
  NodeCount = 2
  (1..NodeCount).each do |i|
    config.vm.define "opensusevm#{i}" do |node|
      node.vm.hostname = "opensusevm#{i}"
      node.vm.box = "opensuse/Leap-15.2.x86_64"
      node.vm.network "private_network", ip: "192.168.56.4#{i}"
      # VirtualBox provider parameters 
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = 2
        vb.customize ["modifyvm", :id, "--ioapic", "on"]
      end
    end
  end
end
