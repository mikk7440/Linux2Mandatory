# Linux2Mandatory
Af Rasmus og Mikkel

#Creating container
$ lxc-create -n R1 -t download -- -d alpine -r 3.4 -a armhf
$ lxc-start -n C1
$ lxc-attach -n C1
Update package list and install needed packages
$ lxc-attach -n C1 -- apk update
$ lxc-attach -n C1 -- apk add lighttpd php5 php5-cgi php5-curl php5-fpm
Uncomment the include "mod_fastcgi.conf" line in /etc/lighttpd/lighttpd.conf
Start the lighttpd service
$ rc-update add lighttpd default
$ openrc
