# Linux mandatory 2
af Rasmus H. og Mikkel L

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
