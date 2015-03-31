```console
$ vagrant ssh
vagrant@lucid64:~$ date
```

If `date` is showing the wrong timezone:

```console
vagrant@lucid64:~$ sudo dpkg-reconfigure tzdata

Current default time zone: 'Europe/London'
Local time is now:      Tue Jul  9 13:21:14 BST 2013.
Universal Time is now:  Tue Jul  9 12:21:14 UTC 2013.

vagrant@lucid64:~$ sudo vim /etc/default/locale

LANG=en_GB.UTF-8
LC_ALL=en_GB.UTF-8

vagrant@lucid64:~$ sudo dpkg-reconfigure locales
vagrant@lucid64:~$ sudo reboot now
```

...and if you also need to manually sync time rather than waiting for `ntpd` to drift it:

```bash
sudo service ntp stop
sudo ntpdate pool.ntp.org
sudo service ntp start
```