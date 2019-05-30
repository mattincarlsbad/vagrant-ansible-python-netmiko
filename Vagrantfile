eos_image = "vEOS-lab-4.20.9M-virtualbox.box"
eos_image_location = "http://your-url-goes-here.com/vEOS-lab-4.20.9M-virtualbox.box"

Vagrant.configure(2) do |config|

# Ubuntu 16.04
  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "ubuntu/xenial64"
    ansible.vm.network "private_network", virtualbox__intnet: "10.0.0.0/24", auto_config: false
    ansible.vm.provision "shell", inline: <<-SHELL
     sudo ifconfig enp0s8 10.0.0.20/24
     sudo apt-get update
     sudo apt-get install git vim python-netaddr python-pip sshpass build-essential libssl-dev libffi-dev python-dev python3-pip python-paramiko -y
     sudo pip install ansible
     sudo pip3 install netmiko
     sudo pip3 install cryptography==2.4.2
     git clone https://github.com/mattincarlsbad/python-netmiko-show
    SHELL
  end

# Ubuntu 18.04
  config.vm.define "ansible2" do |ansible2|
    ansible2.vm.box = "ubuntu/bionic64"
    ansible2.vm.network "private_network", virtualbox__intnet: "10.0.0.0/24", auto_config: false
    ansible2.vm.provision "shell", inline: <<-SHELL
     sudo ifconfig enp0s8 10.0.0.21/24
     sudo apt-get update
     sudo apt-get install git vim python-netaddr python-pip sshpass python3-pip python-paramiko -y
     sudo pip install ansible
     sudo pip3 install netmiko
     sudo pip3 install cryptography==2.4.2
     git clone https://github.com/mattincarlsbad/python-netmiko-show
    SHELL
  end

  config.vm.define "leaf1" do |leaf1|
    leaf1.vm.box = eos_image
    leaf1.ssh.insert_key = false
    leaf1.vm.box_url = eos_image_location
    leaf1.vm.boot_timeout = 3000
    # Internal networks for swp* interfaces.
    leaf1.vm.network "private_network", virtualbox__intnet: "10.0.0.0/24", auto_config: false
    leaf1.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
    # Enable eAPI in the EOS config
    leaf1.vm.provision 'shell', inline: <<-SHELL
      FastCli -p 15 -c "configure
      username admin privilege 15 role network-admin secret admin
      hostname leaf1
      management api http-commands
      no shutdown
      interface eth1
      no switchport
      ip address 10.0.0.1 255.255.255.0
      exit"
    SHELL
  end

  config.vm.define "leaf2" do |leaf2|
    leaf2.vm.box = "CumulusCommunity/cumulus-vx"
    # Internal network for swp* interfaces.
    leaf2.vm.network "private_network", virtualbox__intnet: "10.0.0.0/24", auto_config: false
    #set network interfaces to vbox promiscuous mode
    leaf2.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
    leaf2.vm.provision "shell", inline: <<-SHELL
      sudo net add interface swp1 ip address 10.0.0.2/24
      sudo net commit
    SHELL
  end

end
