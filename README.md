# Linux mandatory 2
by Rasmus H and Mikkel L

A pie zero with desbian hosting a lighttpd server running from a lxc alpine container,  
fetching random numbers from another lxc alpine container.

The lighttpd container will be named M1 and serve on port 80,
the random generator container will be named R2 and serve on port 8080.

## lxc and lxc-net
Inspiration from: https://angristan.xyz/setup-network-bridge-lxc-net/ 

Install lxc and dnsmasq-base, which is used by lxc-net
```sh
$ apt install lxc
$ apt install dnsmasq-base
```
### Configure bridge
Since we want the containers to communicate we set up a bridge.  
We set up the bridge by writing two files in the host file system.
We use vim and nano to create and edit files.
First the configuration file *" /etc/lxc/default.conf "*:  
```sh 
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```
and tell LXC to use the bridge by creating *" /etc/default/lxc-net "* and writting:
```sh
USE_LXC_BRIDGE="true"
```
### Static ip
To be able to access M2 reliably from M1 and M1 from the host, we setup static ips by creating the file *" /etc/lxc/dhcp.conf *". We had a problem with the containers not getting the correct ip's again after restarting the containers, so we added a lease time of 0, which fixed the problem.
```
dhcp-host=M1,10.0.3.21,0
dhcp-host=R2,10.0.3.22,0
```

### Unpriviliged containers
To be able to make unpriviliged containers, we create *" ~/.config/lxc/default.conf "*
```
lxc.id_map = u 0 100000 65536
lxc.id_map = g 0 100000 65536
lxc.network.type = veth
lxc.network.link = lxcbr0
```
and append the following to *" /etc/lxc/lxc-usernet "*
```
pi veth lxcbr0 2
```
### Port forwarding
To be able to access M1:80 from outside the host, we forward port 80 on the host to port 80 on M1:
```sh
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j DNAT --to-destination 10.0.3.21:80
```
## Hosting container M1
Create, start and attach to M1:
```sh
$ lxc-create -n M1 -t download -- -d alpine -r 3.4 -a armhf
$ lxc-start -n M1
$ lxc-attach -n M1
```
Attached to M1, we update the package list and install the needed packages for a php lighttpd server:    
```sh
# apk update
# apk add lighttpd php5 php5-cgi php5-curl php5-fpm
```
Still attached to M1, we uncomment the include "mod_fastcgi.conf" line in /etc/lighttpd/lighttpd.conf
, add the lighttpd service to the default run revel and starts the service:
```sh
# rc-update add lighttpd default
# openrc
```
In M1, create *" /var/www/localhost/htdocs/index.php "* and add (Credit Tórur)
```sh<!DOCTYPE html>
<html><body><pre>
<?php 
        // create curl resource 
        $ch = curl_init(); 
        
        // set url 
        curl_setopt($ch, CURLOPT_URL, "10.0.3.22:8080"); 
        
        //return the transfer as a string 
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); 
        
        // $output contains the output string 
        $output = curl_exec($ch); 
        
        // close curl resource to free up system resources
        curl_close($ch);
        print $output;
?>
</pre></body></html>
```

## Random number container
We exit M1 and create, start and attach to another container, R2, like we did for M1.
Inside the random container M2, we install socat:
```sh
# apk update
# apk add socat
```
We create a script for generating 16 4-byte random numbers *"/bin/rng.sh"*.
We had a problem with random not providing numbers, so we used urandom.
```sh
#!/bin/ash

dd if=/dev/urandom bs=4 count=16 status=none | od -A none -t u4
```
We then serve the script by:
```sh
# socat -v -v tcp-listen:8080,fork,reuseaddr exec:/bin/rng.sh
```

## Conclusion
We are able to generate random numbers in one container, serve them as html by another container and hosting the webpage on the host's port 80.
