---
layout: post
title: High Availability Using Keepalived
subtitle: Part 1\: Mock environment
background: '/img/posts/03-magic.jpg'
---

## What is a Floating IP?

An IP address that can point to different servers based on certain criteria.

## Why do I care?

**High Availability**. Imagine you're serving an application on `190.100.100.100` and you have an entry on `mydns.com` pointing to that IP. You can have 2 of those services running on two different machines and `190.100.100.100` as a floating IP pointing to both services in active / passive fashion.


Ok, machines fail. If you ever administered anything for more than 5 minutes you already know this.
We address this by providing high availability on our services. 

> High availability means no single point of failure for our service.


We'll try to go from this:

![Low availability](https://i.imgur.com/U8IdGjK.png){:height="100%" width="100%"}

To this:

![High availability](https://i.imgur.com/MqRoyMU.png){:height="100%" width="100%"}

What this diagram tries to show is that, if we loose `190.100.100.101` we still redirect traffic, as 190.100.100.100 will attach to `190.100.100.102`.



## Working Example

In this part of the post, we'll create two virtual VMs with a shared folder, both running the same service and storing the same data. We'll then configure a floating IP to point to the main server with keepalived and we'll keep the secondary waiting to take control of the floating IP.

### Setup virtual infrastructure.

We'll be using Vagrant to start two VMs. Vagrant is my favourite way to define VMs as code (keeping your Infrastructure as Code has an enormous amount of benefits, I'd probably post about that someday).

You'll need Vagrant installed on your computer. We're going to use the following Vagranfile to launch two VMs:

```
servers=[
    {
      :hostname => "master-rancher",
      :ip => "192.168.0.100",
      :ip_int => "10.0.2.10",
      :box => "minimum/centos-7-docker",
      :ssh_port => 3300,
      :ram => 1024,
      :cpus => 1,
      :port_range => (3301..3309)
    },
    {
      :hostname => "minion-rancher",
      :ip => "192.168.0.110",
      :ip_int => "10.0.2.11",
      :box => "minimum/centos-7-docker",
      :ssh_port => 3310,
      :ram => 1024,
      :cpus => 1,
      :port_range => (3311..3320)
    }
  ]
$script = <<-SCRIPT
echo "[ INFO ] Disabling firewalld"
systemctl stop firewalld
systemctl disable firewalld
SCRIPT

  Vagrant.configure(2) do |config|
    servers.each do |machine|
      config.vm.define machine[:hostname] do |node|
        node.vm.box = machine[:box]
        node.vm.usable_port_range = machine[:port_range]
        node.vm.hostname = machine[:hostname]
        node.ssh.private_key_path = ["~/.vagrant.d/insecure_private_key"]
        config.ssh.insert_key = false
        config.vm.provision "shell", inline: $script
        node.vm.network "public_network", ip: machine[:ip], bridge: 'en0: Wi-Fi (AirPort)'
        config.vm.network "private_network", type: "dhcp"
        node.vm.network :forwarded_port, guest: 22, host: machine[:ssh_port], id: 'ssh'
        node.vm.synced_folder "/opt/rancher_data", "/rancher_data", 
                              type: "nfs",
                              nfs_version: 3,
                              mount_options: ['noac']

        node.vm.provider "virtualbox" do |vb|
          vb.gui = false
          vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
          vb.customize ["modifyvm", :id, "--cpus", machine[:cpus]]
          vb.name = machine[:hostname]
          for i in machine[:port_range]
            node.vm.network :forwarded_port, guest: i, host: i
          end
        end
      end
    end
  end
  
```

Save that file as `Vagrantfile` and issue `vagrant up` from the same directory as that file.

You might nee to change the `bridge: 'en0: Wi-Fi (AirPort)'` to whatever yours is (you can try running this as is and selecting the correct one from the list that vagrant will provide).

I'm specifying the IPs as 192.168.0.100 for the master and 192.168.0.110 for the minion. Note also we're defining an "nfs" for the machines, to have a common data store.

I have an ansible role I need to publish for this. But in the meantime, just login `vagrant ssh master-rancher` and `vagrant ssh minion-rancher`.

Enter this commands for both machines (tmux could come in handy here if you know what I mean). If you get asked for a password `vagrant` is the default one.

This will get docker running inside the VMs
```{bash}
sudo yum update
sudo yum install -y docker
```

You now need to start an instance of rancher in each of the VMs:

```

docker run -p 3300:443 \
  --name rancher-$(hostname) \
  -v /opt/rancher_data:/var/lib/rancher
  rancher:latest
        
```

This will get our two services running on two separated machines. We'll configure the `keepalived` part on the next post.