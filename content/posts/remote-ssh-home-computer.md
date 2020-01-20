+++
title = "Cloud is not free, Wakeup and SSH to your home computer remotely"
date = "2019-03-10T17:04:04-05:00"
tags = [
    "ssh",
    "cloud"
]
categories = [
    "setup",
]
+++

![teepublic](https://cdn-images-1.medium.com/max/1260/1*SNdOlOGAYF6ahI1SG22yVA.jpeg " ")

Recently I had to run some [Q-learning](https://medium.com/@kyle.jinhai.li/reinforcement-learning-introduction-to-q-learning-444c951e292c) algorithms on my home desktop (running on Ubuntu 18). These workloads, depends on the model and hyper parameters, could take a while to finish. If I’m away from home and still wants to work on this project, my options would be:

- Run the algorithms on my Macbook pro and thus starting a fire somewhere

- Run the algorithms on the cloud, cheapest [computation optimized instance](https://aws.amazon.com/ec2/pricing/on-demand/) costs \$0.085/hour on AWS

- SSH into my desktop remotely, and continue my workloads, for free (well, plus the electricity cost)

In the rest of this post I will walk through how I set up my desktop for SSH and WOL(Wakeup On LAN).

# SSH within the same local network

1. Find the ip of destination machine: run `ifconfig` and you want to find the interface with your local ip, something like `192.168.0.10`

1. Verify `sshd` service is running and find the ssh port (default is 22)

1. Test ssh: `ssh user@192.168.0.10`

# SSH from external network

In order to access the desktop from external network, we need to:

1. Find the temporary public ip assigned by the ISP, `dig +short myip.opendns.com @resolver1.opendns.com` let’s say `1.2.3.4` is our public ip

1. Now that we have the public ip, can we ssh to our desktop remotely? Well, not yet, our home devices all share the same public ip, this is called [Network Address Translation](https://en.wikipedia.org/wiki/Network_address_translation) (NAT). So we need to explicitly tell our router to forward all incoming traffic of a port to a specific local ip and port, this is called Port Forwarding. In our case we need to tell our router to forward all traffic heading to port 22 of our desktop. All routers should have this functionality and this can be setup on the router admin web ui. We need to add a port forwarding rule so that it forwards source port `22` to destination, our desktop’s ip, `192.168.0.10`and destination port `22.`

1. Test ssh, `ssh user@1.2.3.4`

1. Tighten up the security. Internet is a dangerous place and we don’t want our machine compromised. In my setup I disabled password authentication and root login, so that the only way to ssh into my desktop is through public key authentication. To disable password authentication, in `/etc/ssh/sshd_config`, change the following lines to `no`

```
PermitRootLogin no
PasswordAuthentication no
```

5. Before we restart sshd and apply the changes, we need to [create a ssh key pair](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and copy our public key to user’s `authorized_keys`: `ssh-copy-id user@1.2.3.4`

6. Now restart sshd on desktop and test ssh again, this time it should not prompt for password.

# Setting up Wakeup On LAN (WOL)

I set my desktop to suspend after 15mins of idle to save electricity. But the problem is that we can’t ssh back to it after it goes to sleep. The router sees the device is offline and stop forwarding anything to that local ip. Luckily, there is a feature called Wakeup On LAN that’s supported by most motherboards.

1. This feature is usually disabled by default, we need to [enable it in our BIOS](https://www.howtogeek.com/70374/how-to-geek-explains-what-is-wake-on-lan-and-how-do-i-enable-it/)

1. Enable WOL with `ethtool` in the OS level, `sudo ethtool -s eth0 wol`

1. Verify WOL is enabled for our network interface `eth0`. `g` indicates wake on Magic Packet is enabled

```
sudo ethtool eth0

Settings for eth0:
 	Supported ports: [ MII ]
 	Supported link modes:   10baseT/Half 10baseT/Full
 	...
         ...
 	Supports Wake-on: g
 	Wake-on: g
 	Link detected: yes
```

3. Now we need something to send the magic packet, I use [wakeonlan](https://github.com/jpoliv/wakeonlan) but there are many alternatives. To send the wake up our desktop, we need the mac address of our desktop network card. This can be found in the output of `ifconfig` :

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.10  netmask 255.255.255.0  broadcast 192.168.100.255
        inet6 fe80::edf:c31e:840e:92ca  prefixlen 64  scopeid 0x20<link>
       ** **ether** e0:d5:5e:29:28:2b**  txqueuelen 1000  (Ethernet)
        RX packets 26988  bytes 6051736 (6.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5053  bytes 582242 (582.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16  memory 0xef200000-ef220000
```

This tool basically boardcast to all devices under the same network and wakes up the device that matches the mac address

4. Suspend the desktop and test WOL from laptop

```
wakeonlan e0:d5:5e:29:28:2b
Sending magic packet to 255.255.255.255:9 with e0:d5:5e:29:28:2b
```

I can hear my desktop spinning up right after sending this command, we can ssh again! But wait, can we run wakeonlan from an **external network**? How do we wake up the desktop **remotely**?

Unfortunately most home routers do not support this feature, as the above ARP query only works within the local network. This left us a few options:

- Leave the desktop on 24/7 (this might be more expansive than cloud)

- Running a device with low power consumption 24/7 and use it as a jump host to wake up our desktop

- Run [Teamviewer WOL](https://community.teamviewer.com/t5/Knowledge-Base/How-does-Wake-on-LAN-WOL-with-TeamViewer-work/ta-p/33) if the target computer is connected to the router via a network cable.

Luckily I have a raspberry pi sitting around, we can follow the same steps above to setup port forwarding and ssh. Note that we already used port 22 for our desktop, we need to run sshd on a different port, so that we can ssh into raspi from external network. To do this, change `Port 22` to `Port 2223`in `/etc/ssh/sshd_config` and restart sshd.

Then setup another port forwarding rule to forward all incoming traffic to port 2223 to raspi’s ip and port 2223

5. Test ssh to raspi using our public ip and test wakeonlan from raspi

```
ssh piuser@**1.2.3.4** -p 2223

wakeonlan e0:d5:5e:29:28:2b
Sending magic packet to 255.255.255.255:9 with e0:d5:5e:29:28:2b
```

Nice, desktop is up again! We can now wake up and ssh into our computer remotely.

# Future Enhancement

Our public ip is not permanent, it could be changed when the DHCP lease is renewed. We can add a cron job on our raspi to commit our public ip to a private git repo if ip is changed.
