---
title: "Vagrant Multi machine"
date: 2018-09-09T20:06:59+02:00
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


The official documentation can be found [here](https://www.vagrantup.com/docs/multi-machine/)