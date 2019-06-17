---
layout: post
title: High Availability Using Keepalived
subtitle: Part 2 Keepalived Configuration
background: '/img/posts/03-magic.jpg'
---

## What is a Keepalived

A linux service that can be used to manage a floating IP. This is an IP address that can point to different servers based on certain criteria.

## Why do I care?

**High Availability**. Imagine you're serving an application on `190.100.100.100` and you have an entry on `mydns.com` pointing to that IP. You can have 2 of those services running on two different machines and `190.100.100.100` as a floating IP pointing to both services in active / passive fashion.


In this part I'll explain how to configure Keepalived to manage this situation, having a service already running on two different machiens (covered in Part1).

We'll try to go from this:

![Low availability](https://i.imgur.com/U8IdGjK.png){:height="100%" width="100%"}

To this:

![High availability](https://i.imgur.com/MqRoyMU.png){:height="100%" width="100%"}

What this diagram tries to show is that, if we loose `190.100.100.101` we still redirect traffic, as 190.100.100.100 will attach to `190.100.100.102`.

## Install `keepalived`
We now need to abstract the virtual IP from the machines and point depending on which one is available.

For this, as we mentioned before, we'll use `keepalived`.

We first install keepalived on both servers:

```
yum install -y keepalived
``` 

## Create the condition for keepalived

To create a condition in which keepalived will base the attachment of the ip to one machine or the other, we will create a script that have a return code 0 (success) if certain criteria is met. 

The script returns 0 in this example if the service we try to set in HA (rancher) is alive and a different exit code if it doesn't. We store this script on `/opt/check_rancher.sh`:

```
curl -k -s --head  --request GET https://localhost:3300 && \
          (echo "rancher active" && exit 0) || \
          (echo "rancher not active" && exit -1)
```


We will now configure the two keepalive instance, on the master and on the slave. To do so, we need to include the following on the `/etc/keepalived/keepalived.conf`:


For the master:

```
global_defs {
# This should be different for each of the keepalived.
  router_id rancher_master
}
# Script used to check if Rancher is running
vrrp_script check_rancher {
  script "/opt/check_rancher.sh"
  interval 2
  weight 2
}
# Virtual interface
# The priority specifies the order in which the assigned interface to take over in a failover
vrrp_instance VI_01 {
  state MASTER
  # You need to change this to whatever your if is (check with ifconfig)
  interface enp0s8
  virtual_router_id 51
  # Give the master node Higher priority here
  priority 101
  # The virtual ip address shared between the two loadbalancers
  virtual_ipaddress {
    190.100.100.100
  }
  track_script {
    check_rancher
  }
}
```

For the minion (same file on the machine running the backup service):

```
global_defs {
# This should be different for each of the keepalived.
  router_id rancher_minion
}
# Script used to check if Rancher is running
vrrp_script check_rancher {
  script "/opt/check_rancher.sh"
  interval 2
  weight 2
}
# Virtual interface
# The priority specifies the order in which the assigned interface to take over in a failover
vrrp_instance VI_01 {
  state MINION
  # You need to change this to whatever your if is (check with ifconfig)
  interface enp0s8
  virtual_router_id 51
  # Give the master node Higher priority here
  priority 100
  # The virtual ip address shared between the two loadbalancers
  virtual_ipaddress {
    190.100.100.100
  }
  track_script {
    check_rancher
  }
}
```

We now restart `keepalived` on both machines:

```
service keepalived restart
```

We should be seeing now the IP redirecting to the master node if /opt/check_rancher.sh returns zero.

To debug possible keepalived issues you can always:

```
journalctl -u keepalived.service --follow
```

