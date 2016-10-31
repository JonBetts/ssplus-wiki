**These have been copied form the l92 instructions so may need tweaking.**

Ok, you’ve setup your server and it’s either hosting sites, database or both. You’ll need to back that up somehow, and our default option is to backup Oracle databases using rman, and to rsync uploaded files offsite to our server pandora

To back up the Oracle database(s) on your server, see the instructions in the ssplus repo

Here’s how to set up rsync for uploaded files (under /sites):

If there isn’t one already, create a backup user and backup folder

$ `sudo useradd -m -s /bin/bash -G oinstall backup`

$ `sudo mkdir /var/backup`

$ `sudo chown backup:backup /var/backup`

One already exists on ubuntu installs, so you’ll probably just want to mod that account and allow write access for a couple of folders

$ `sudo usermod -s /bin/bash -G oinstall backup`

$ `cd /var/backups`

$ `sudo mkdir -p .ssh bin`

$ `sudo chown backup:backup .ssh bin`

$ `sudo chmod 700 .ssh`

$ `sudo vim /sites/.rsync-filter`
and paste-in these lines (to exclude the _old folder, and other folders which don’t really need backing-up)

- _old
- deploy_backup
- logs
- *.war
- webapps
- temporary.maintenance
Now set up ssh access from pandora:


$ `sudo passwd backup`

$ `sudo su - backup`

$ `mkdir -p .ssh bin`

$ `echo "insert public key for pandora backup user here" > .ssh/authorized_keys`

$ `chmod 700 .ssh`

$ `chmod 600 .ssh/authorized_keys`

$ `Now log-in to pandora…`

Start by ensuring your host is in /etc/hosts then initialise the ssh connection (this serves as a test but also adds the host’s key to known_hosts), e.g.

$ `sudo su - backup`
$ `ssh keaton`
$ `logout`
Then simply add the host into ~/backup.sh to the list of site hosts.