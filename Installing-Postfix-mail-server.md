--Already applied via chef

First of all, add the server’s IP address to the list of Outbound mail servers under the Connections tab of the DEFAULT policy in the Email Security section of the Websense portal

`sudo apt-get install postfix bsd-mailx`
To use a relay or ‘smart’ host, specify cust32998-s.out.mailcontrol.com as the relayhost (if not done during installation prompts, you can edit `/etc/postfix/main.cf` and add the line `relayhost = cust32998-s.out.mailcontrol.com`, then restart postfix).

Without a relayhost, ensure myhostname matches the reverse DNS entry for the server’s IP address, and set `smtp_tls_security_level = may` to use opportunistic email encryption.

In either case, it also helps a lot if DNS resolution is fast. Check this using nslookup. If you need to, change the nameservers in `/etc/resolv.conf` and, on Ubuntu, in `/etc/network/interfaces`. Options might be OpenDNS or Google Public DNS.

You will probably also want to add some mail aliases: 
`sudo vim /etc/aliases`
and add entries accordingly. Postfix usually gives you 
See man 5 aliases for format
`postmaster: root`

and I usually add
Person who should get root's mail
`root: tech_team`

redirections of other users' mail
`backup: tech_team`
`ben: ben.dilley@skillstream.co.uk`

external email addresses
`tech_team: technical@skillstream.co.uk`

to apply changes to `/etc/aliases` you then need to run

`sudo newaliases`