Do this as early as possible in the OS setup process: the more software you add the more likely it is that changing the hostname will be non-trivial.

The simplest version:

`sudo hostname mynewhostname`

`sudo hostname > /etc/hostname`

`sudo vim /etc/hosts`

...where editing /etc/hosts is so that any reference to the previous hostname is changed to the new hostname.

On centos/redhat/fedora I found that it was necessary to edit /etc/sysconfig/network to convince the hostname to stick on reboot. Here, the HOSTNAME must be the fully-qualified domain name (FQDN), preferably the one that has a DNS A record and reverse-DNS for the external IP address.