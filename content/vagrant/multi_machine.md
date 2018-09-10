---
title: "Vagrant Multi machine"
date: 2018-09-09T19:06:59+02:00
order: 1
---

# Multiple machines

You can configure a `Vagrantfile` for multiples VMs at the same time.  


You need to use the function `vm.define` is the main VM configuration like the following configuration.
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "echo Common provisioning"

  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/xenial64"
    web.vm.provision "shell", inline: "echo Provisioning for Web only"
  end

  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"
    db.vm.provision "shell", inline: "echo Provisioning for Db only"
  end
end
```

Try to run this configuration.  
You should see two VMs starting each provisionning the common part and it's custom part.

By default, your `vagrant ssh` will connect to the main vm (the first configured).  
You can change it with :
```ruby
config.vm.define "web", primary: true do |web|
  # ...
end
```

You can also choose the connection VM with `vagrant ssh db`

The official documentation can be found [here](https://www.vagrantup.com/docs/multi-machine/)

# Excercice

Create a `Vagrantfile` to create 2 VMs, one with a `mysql` service installed and `mysql-client` tool on the other.
Try to connect on the VM with the client and connect to the other VM with the Mysql server.