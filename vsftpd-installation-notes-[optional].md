(redhat)
$`sudo up2date -i vsftpd`

or (ubuntu/debian)
$`sudo apt-get install vsftpd`

or (fedora)
$`sudo yum install vsftpd`

$`sudo vim /etc/vsftpd/vsftpd.conf`

change anonymous_enable to NO and add/uncomment the lines 

`pasv_max_port=51000
pasv_min_port=50000
pasv_enable=YES
local_enable=YES
chroot_local_user=YES
enable writing if required 
write_enable=YES`

$`sudo vim /etc/sysconfig/iptables` (yes, I know you’re not supposed to edit directly)
add these two rules for connection and passive modes respectively (haven’t figured out active mode requirement yet)

`-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT`

`-A INPUT -m state --state ESTABLISHED -m tcp -p tcp --dport 50000:51000 -j ACCEPT`

$`sudo vim /etc/sysconfig/iptables-config`
add ip_conntrack_ftp to the IPTABLES_MODULES list so it looks like e.g. 
IPTABLES_MODULES="ip_conntrack_netbios_ns ip_conntrack_ftp"
...or if there is no iptables-config, you can load the ip_conntrack_ftp module on boot by adding it to /etc/modules

$`sudo vim /etc/modprobe.conf`
add the following line: 
options `ip_conntrack_ftp ports=21,50000:51000 > sudo /etc/init.d/iptables restart > sudo /etc/init.d/vsftpd start`
To keep it online after a reboot (red hat/fedora): 
$`chkconfig vsftpd on`