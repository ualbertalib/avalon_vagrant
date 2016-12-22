# -*- mode: ruby -*-
# vi: set ft=ruby :
# For running Tai-chun's ansible-dev.sh script from the ansible-config repository.

# Instructions:
#   1) Set the names in the inventory-dev files all to avdev01-cwant
#   2) Point avdev01-cwant to the "private_network" address below in /etc/hosts
#   2) Bring up this Vagrant box. Reboot it to enable selinux
#      (ansible will complain if you don't)
#   3) Run the playbook:
#      $ cd ansible-config/projects/
#      $ bash ansible-dev.sh
#      (Wait a really long time)

AVALON_FQDN = 'avdev01-local.library.ualberta.ca'
AVALON_IP = '192.168.33.133'

def public_key_path
  ENV['HOME'] + '/.ssh/id_rsa.pub'
end

def public_key
  file = File.open(public_key_path, "rb")
  contents = file.read
  file.close
  return contents
end

def hostname_fqdn
  AVALON_FQDN
end

def hostname_fqdn_escaped
  hostname_fqdn.gsub('.', '\.')
end

def hostname_short
  hostname_fqdn.split('.')[0]
end

def avalon_ip
  AVALON_IP
end

Vagrant.configure(2) do |config|
  config.vm.box = "ual/centos7.0"
  config.vm.box_url = "http://129.128.46.152/vagrantboxes/centos70.json"
  config.vm.network "private_network", ip: avalon_ip
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.network "forwarded_port", guest: 443, host: 8443

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "8192"
     vb.cpus = 4
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Configure some repos (get new ruby, old mediainfo)
    yum -y remove ualib-custom7
    yum -y remove ualib-custom6
    rpm -i \
      http://129.128.216.203/custom/rhel/7/test/x86_64/ualib-testcustom7-1.8-0.el7.x86_64.rpm

    # Bug in recent mediainfo version

    yum install yum-plugin-versionlock

    yum -y install libmediainfo-0.7.86-2.1 mediainfo-0.7.86-2.1
    yum versionlock mediainfo libmediainfo

    # wget is just installed for testing
    yum -y install ansible python-setuptools firewalld wget

    # Inject root pubkey
    echo "#{public_key.strip}" >> /root/.ssh/authorized_keys
    service firewalld start
    chkconfig firewalld on

    # Note a reboot is needed before selinux will truly be enabled (in permissive mode)
    sed -i -e 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

    # Update all packages
    yum update -y

    # Configure hostnames (short and FQDN)
    echo avdev01-local > /etc/hostname
    sed -i -e 's/^127\.0\.0\.1\s*localhost/127\.0\.0\.1   #{hostname_fqdn_escaped} #{hostname_short} localhost/' /etc/hosts
  SHELL
end
