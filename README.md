# avalon_vagrant
This contains the Vagrant file and documentation that I use to set up an Avalon dev stack using VirtualBox and our local ansible playbook from the ansible-config repository.

## Instructions:

### Environment

1. Select a place where you would like to store a virtual base
     box for provisioning VMs. Put the path to this directory in
     the environment variable LOCAL_BOX_PATH. If not set, this
      will default to "/scratch/dont_backup/cwant/avalon/vagrant_boxes".

### Create the base box package

We do this to reduce the time of reprovisioning.

1. Build the box: <pre>$ vagrant up avalon-base-box</pre>
2. Check the number of cpus that get provisioned in this vagrant box, modify
     as necessary. This check is done by ssh-ing and checking the contents
     of /proc/cpuinfo. The fix is to modify the settings in VirtualBox
     directly via the user interface.
3. Package the box:<pre>
$ vagrant package avalon-base-box --output $LOCAL_BOX_PATH</pre>

This base box needs to be created once, and can be used to
create all of the derivative boxes

### Provision the derivative you need from the base.

1. Easy, e.g., <pre>$ vagrant up avdev01-local</pre>
2. Most of the provisioning work was done in the setup for the base box,
   so this step mostly just sets up the hostname for the box. This can require
   a reboot to fully work, this is done via: <pre>$ vagrant reload avdev01-local</pre>

### Install the app via ansible, e.g. for avdev01-local

1. checkout the 'ansible-config' repository, and set the vault password
   in ~/.vault (ask a staff member for the password, and assistance
   checking out the private repository).
2. Set the names in the inventory-dev files all to avdev01-local,
   and any FQDN related names to avdev01-local.library.ualberta.ca
3. Point avdev01-local to the "private_network" address in the
   Vagrantfile in your host /etc/hosts. For the host avdev01-local, this should
   be 192.168.33.133, e.g., add to /etc/host:<pre>
   192.168.33.133  avdev01-local avdev01-local.library.ualberta.ca
</pre>
4. Bring up this Vagrant box. Reboot it to enable selinux
   (ansible will complain if you don't)
5. Run the playbook:<pre>
 $ cd ansible-config/projects/
 $ bash ansible-dev.sh
 </pre> (Wait a really long time)

### Log into your development box, and access it via a browser

Part of the provisioning process was to inject your public key into the root
account. So there are two ways in which you can access your box via SSH:

<pre>$ ssh root@avdev01-local</pre>
or
<pre>$ ssh deploy@avdev01-local</pre>
or
<pre>$ vagrant ssh avdev01-local</pre>

The last method connects you to the vagrant account, which has sudo priviledges.

The avalon web app is available by pointing your browser to:

<pre>https://avdev01-local.library.ualberta.ca</pre>

You can log into the admin account at the page:

<pre>https://avdev01-local.library.ualberta.ca/users/auth/identity</pre>

No direct link points to this page. The login id is avalondev@library.ualberta.ca,
and the password can be obtained by viewing one of the vaulted files from the ansible
playbook: <pre>$ ansible-vault view roles/avalon_app/vars/dev.yml
</pre>

(This requires the vault password you obtained earlier.)

You can run the test suite by logging into the deploy account:
<pre>
$ ssh deploy@avdev01-local
$ cd /var/www/avalon/
$ bundle exec rake spec
</pre>
