# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.

  config.vm.box = "ubuntu/trusty64"

  # config.vm.box_check_update = false

 web_ip = "192.168.33.10"

 # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  (1..1).each do |i|
      config.vm.define "haproxy-#{i}" do |node|
        http_port  = (i + 7) * 1000 + 80
        admin_port = (i + 7) * 1000 + 404
        https_port = (i + 7) * 1000 + 443
        octet = i + 10
        node.vm.hostname = "haproxy-#{i}"
        node.vm.network "forwarded_port", guest: 80,   host: http_port
        node.vm.network "forwarded_port", guest: 8404, host: admin_port
        node.vm.network "forwarded_port", guest: 443,  host: https_port

        node.vm.network "private_network", ip: "192.168.33.#{octet}"
        node.vm.provision "shell", inline: <<-SHELL
          sudo locale-gen de_DE.UTF-8
          sudo add-apt-repository ppa:vbernat/haproxy-1.5
          sudo apt-get update
          sudo apt-get install -y haproxy hatop
          sudo cp /vagrant/configs/haproxy.cfg /etc/haproxy
          sudo cp /vagrant/configs/rsyslog.conf /etc/
          sudo echo "#{web_ip} web be01.example.com be02.example.com be03.example.com be01 be02 be03" >> /etc/hosts
          sudo mkdir -p /etc/haproxy/ssl/certs
          sudo mkdir -p /etc/haproxy/ssl/ca
          sudo cp /vagrant/ssl/*_combined.crt /etc/haproxy/ssl/certs
          sudo cp /vagrant/ssl/ca.crt /etc/haproxy/ssl/ca
          sudo service rsyslog restart
          sudo service haproxy restart
        SHELL

    end
  end
  config.vm.define "web" do |web|
      web.vm.hostname = "web"
      web.vm.network "private_network", ip: "192.168.33.10"
      web.vm.provision "shell", inline: <<-SHELL
        sudo locale-gen de_DE.UTF-8
        sudo apt-get install -y apache2
        sudo a2enmod headers
        sudo a2dissite 000-default
        cd /vagrant/configs
        for x in site-*.conf; do
          sudo cp ${x} /etc/apache2/sites-available
          sudo a2ensite ${x%.*}
        done
        sudo cp site-*.conf /etc/apache2/sites-available
        sudo cp ports.conf /etc/apache2/
        sudo cp apache2.conf /etc/apache2
        sudo echo "#{web_ip} be01.example.com be01 be02.example.com be02 be03.example.com be03" >> /etc/hosts
        sudo service apache2 restart
      SHELL
  end

end
