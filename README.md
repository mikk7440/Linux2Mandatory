# Linux mandatory 2
af Rasmus H. og Mikkel L

# Table of content
- [Install lxc-net](##Install%20lxc-net)
- [Create containers](##Create%20containers)

<!-- toc -->

## Install lxc-net
blabla:Biiiig inspiration from: https://angristan.xyz/setup-network-bridge-lxc-net/ 

```sh
$ apt install dnsmasq-base

$ systemctl restart lxc-net
$ systemctl status lxc-net
```
### Configure bridge
something bridge  
Make sure *" /etc/lxc/default.conf "* only contains the following  
```sh
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```
tell LXC to 
```sh
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```

## Create containers
```sh
$ lxc-create -n C1 -t download -- -d alpine -r 3.4 -a armhf
$ lxc-start -n C1
$ lxc-attach -n C1
```
Update package list and install needed packages  
```sh
$ lxc-attach -n C1 -- apk update
$ lxc-attach -n C1 -- apk add lighttpd php5 php5-cgi php5-curl php5-fpm
```
Uncomment the include "mod_fastcgi.conf" line in /etc/lighttpd/lighttpd.conf  
Obs! sometimes this comment itself out again  

Start the lighttpd service  
```sh
$ rc-update add lighttpd default
$ openrc
```
