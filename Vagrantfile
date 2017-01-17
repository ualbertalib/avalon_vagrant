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

AVALON_HOSTS = {
  'avdev01-local' => {
    fqdn: 'avdev01-local.library.ualberta.ca',
    ip: '192.168.33.133',
    cpus: 4,
    mem: 8192
  },
  'avtest01-local' => {
    fqdn: 'avtest01-local.library.ualberta.ca',
    ip: '192.168.33.134',
    cpus: 4,
    mem: 8192
  },
  'hydra-dive-in' => {
    fqdn: 'hydra-dive-in.library.ualberta.ca',
    ip: '192.168.66.134',
    cpus: 4,
    mem: 8192
  },
}.freeze

def public_key_path
  ENV['HOME'] + '/.ssh/id_rsa.pub'
end

def public_key
  file = File.open(public_key_path, "rb")
  contents = file.read
  file.close
  return contents
end

def hostname_fqdn(host)
  host[:fqdn]
end

def hostname_fqdn_escaped(host)
  hostname_fqdn(host).gsub('.', '\.')
end

def hostname_short(host)
  hostname_fqdn(host).split('.')[0]
end

def host_ip(host)
  host[:ip]
end

def provision_string(host)
  return <<-SHELL
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
    echo #{hostname_short(host)} > /etc/hostname
    sed -i -e 's/^127\.0\.0\.1\s*localhost/127\.0\.0\.1   #{hostname_fqdn_escaped(host)} #{hostname_short(host)} localhost/' /etc/hosts
  SHELL
end

Vagrant.configure(2) do |config|
  config.vm.box = "ual/centos7.0"
  config.vm.box_url = "http://129.128.46.152/vagrantboxes/centos70.json"

  AVALON_HOSTS.each do |host_definition, host|
    config.vm.define host_definition do |box|
      box.vm.network "private_network", ip: host[:ip]

      box.vm.provider "virtualbox" do |vb|
        vb.memory = host[:mem]
        vb.cpus = host[:cpus]
      end

      box.vm.provision "shell", inline: provision_string(host)
    end
  end
end
