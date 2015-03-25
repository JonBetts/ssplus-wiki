```console
$ vagrant ssh
vagrant@lucid64:~$ date
```

If `date` is ahowing the wrong timezone:

```console
vagrant@lucid64:~$ sudo dpkg-reconfigure tzdata

Current default time zone: 'Europe/London'
Local time is now:      Tue Jul  9 13:21:14 BST 2013.
Universal Time is now:  Tue Jul  9 12:21:14 UTC 2013.

vagrant@lucid64:~$ sudo vim /etc/default/locale

LANG="en_GB"
LANGUAGE="en_GB:en"

vagrant@lucid64:~$ sudo vim /var/lib/locales/supported.d/local

en_GB ISO-8859-1

vagrant@lucid64:~$ sudo reboot now
```

...and if you also need to manually sync time rather than waiting for `ntpd` to drift it:

```bash
sudo service ntp stop
sudo ntpdate pool.ntp.org
sudo service ntp start
```