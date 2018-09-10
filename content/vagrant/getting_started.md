---
title: "Getting Started with Vagrant"
date: 2018-09-09T14:06:59+02:00
order: 0
---

# Installation

Installation instruction can be found on [this page](https://www.vagrantup.com/downloads.html)

You can verify the installation by running the `vagrant` command.
```bash
▶ vagrant
Usage: vagrant [options] <command> [<args>]

    -v, --version                    Print the version and exit.
    -h, --help                       Print this help.

# ...
```

# First VM

Let's start by crate a VM with `vagrant`.

```bash
▶ vagrant init ubuntu/xenial64
▶ vagrant up
```

The Vagrant command will download a `box` (the vagrant term for an VM image for a provider), create a VM based on this image and start it.  
In our case it will start a VM based on the `Ubuntu Xenial (16.04)` image.  

Connect to to your running instance with `vagrant ssh`
```bash
▶ vagrant ssh
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-134-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

New release '18.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


vagrant@ubuntu-xenial:~$
```
You are now connected to your running VM, you can try to break it for fun.  
Exit when you are ready to continue.

To destroy the running VM `vagrant destroy`

# Vagrantfile

When we ran the `vagrant init ubuntu/xenial64` command, if you looked closely to the command output, you've seen that a `Vagrantfile` have been created in your current directory.  
This file is the configuration file of the VM and it's the information source for the `vagrant` command.  

```ruby
# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/xenial64"

  ...
end
```

The configuration is heavily documented. I invite you to read the configurations availables and the quick explanation.  
The documentation is also available [here](https://docs.vagrantup.com)

# Provisioning

On the last part on the `Vagrantfile` there is a section with an interesting part : the provisionning section.  
Reinstall a VM could be an really easy but boring task to execute on each instanciation.  
The provisionning part will let you define some tasks to be automatically executed by vagrant on the machine startup.  

Here is one exemple of the configuration based on the `shell` inline provisionning :
```ruby
# Enable provisioning with a shell script. Additional provisioners such as
# Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
# documentation for more information about their specific syntax and use.
config.vm.provision "shell", inline: <<-SHELL
  cat "File created by provisioning >> /tmp/witness"
SHELL
```

Try this setup by starting the VM and check the created file : 
```
▶ vagrant up
▶ vagrant ssh
...
vagrant@ubuntu-xenial:~$ cat /tmp/witness
File created by provisioning
```

Another great thing : you can relaunch the provisioning of the VM without recreate it with the command `vagrant up --provision`.


# Exercice

Create a `Vagrantfile` to configure a VM with a fixed ip `10.30.0.1` and a `nginx` http server installed on provisionning.  
You should be able to see the `nginx` homepage at `http://10.30.0.1`