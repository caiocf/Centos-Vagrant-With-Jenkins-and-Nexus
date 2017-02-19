# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos/7"
  config.vm.synced_folder ".", "/vagrant", type:"virtualbox"
  config.vm.network :forwarded_port, guest: 80, host: 8888

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
 
    # Customize the amount of memory on the VM:
    vb.memory = "8192"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    # Setting up password for vagrant user and root
    echo "vagrant" | passwd --stdin vagrant
	echo "vagrant" | passwd --stdin root
	
	# Increasing timeout value due to CGI proxy scanning
	grep timeout /etc/yum.conf > /dev/null || echo timeout=600 | sudo tee -a /etc/yum.conf
	
	# Installing docker as per https://docs.docker.com/engine/installation/linux/centos/
	sudo yum -y update
	sudo yum -y remove docker docker-common container-selinux
	sudo yum -y remove docker-selinux
	sudo yum install -y yum-utils
	sudo yum-config-manager --add-repo https://docs.docker.com/engine/installation/linux/repo_files/centos/docker.repo
	sudo yum makecache fast
	sudo yum -y install docker-engine
	sudo groupadd docker
	sudo usermod -aG docker vagrant
	sudo systemctl enable docker
	
	# Install nginx
	sudo yum -y install epel-release
	sudo yum -y install nginx
	sudo systemctl enable nginx
	semanage permissive -a httpd_t
	
	# Setting up gnome and using that on start
    sudo yum -y groupinstall "GNOME Desktop"
    sudo systemctl set-default graphical.target
    sudo systemctl start graphical.target
	
	# Install Maven
    sudo yum -y install java-1.8.0-openjdk-devel-debug java-1.8.0-openjdk-devel maven
	
	# Install Nexus
	docker pull sonatype/nexus3
	docker rm nexus
	docker run -d -p 48001:8081 --name nexus -e NEXUS_CONTEXT=nexus sonatype/nexus3
	sudo cp /vagrant/dockers/nexus/nginx.conf /etc/nginx/default.d/nexus.conf
	
	# Installing Jenkins
	docker pull jenkins
	docker rm jenkins
	docker run -d -p 49001:8080 --name jenkins -v /vagrant/dockers/jenkins/jenkins_home:/var/jenkins_home:z -t jenkins --prefix=/jenkins
	sudo cp /vagrant/dockers/jenkins/nginx.conf /etc/nginx/default.d/jenkins.conf
	
  SHELL
end
