# Linux mandatory 2
af Rasmus H. og Mikkel L

A pie zero with desbian hosting a lighttpd server running from a lxc alpine container,  
fetching random numbers from another lxc alpine container.

# Table of content
- [Install lxc-net](##Install)
- [Create containers](##Create)

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
Tell LXC to use the bridge by creating *" /etc/default/lxc-net "* and writting:
```sh
USE_LXC_BRIDGE="true"
```

## Create containers
Be sure you can make unprivileged containers by:  

```sh
$ mkdir -p ~/.config/lxc
$ echo "lxc.id_map = u 0 100000 65536" > ~/.config/lxc/default.conf
$ echo "lxc.id_map = g 0 100000 65536" >> ~/.config/lxc/default.conf
$ echo "lxc.network.type = veth" >> ~/.config/lxc/default.conf
$ echo "lxc.network.link = lxcbr0" >> ~/.config/lxc/default.conf
$ echo "$USER veth lxcbr0 2" | sudo tee -a /etc/lxc/lxc-usernet
```
### For each container

Create to containers like the following but with diffrent names, ex. M1 and M2  

```sh
$ lxc-create -n M1 -t download -- -d alpine -r 3.4 -a armhf
$ lxc-start -n M1
$ lxc-attach -n M1
```
Update package list and install needed packages inside the container:    
```sh
# apk update
# apk add lighttpd php5 php5-cgi php5-curl php5-fpm
```
Then uncomment the include "mod_fastcgi.conf" line in /etc/lighttpd/lighttpd.conf  
Obs! sometimes this comment itself out again, then just do this again.  
Start the lighttpd service, inside the container:
```sh
# rc-update add lighttpd default
# openrc
```
Then do this again for container M2.
  
Good stuff:  
/etc/init.d/lighttpd restart  
restart lighthttpd
