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

### Install the app via ansible, e.g. for avdev01-local

1. checkout the 'ansible-config' repository
2. Set the names in the inventory-dev files all to avdev01-local
3. Point avdev01-local to the "private_network" address below in /etc/hosts
4. Bring up this Vagrant box. Reboot it to enable selinux
   (ansible will complain if you don't)
5. Run the playbook:<pre>
 $ cd ansible-config/projects/
 $ bash ansible-dev.sh
 </pre> (Wait a really long time)
