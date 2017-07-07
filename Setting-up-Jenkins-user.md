**This sets up a jenkins users to allow us the ability to use jenkins to deploy sites to servers**
1. `sudo useradd -m jenkins -s /bin/bash`
2. `sudo usermod -aG deploy jenkins`
3. `sudo usermod -aG tomcat jenkins` (where applicable)
4. * `sudo vim /etc/sudoers`, if the sudoers file remains readonly you'll need to edit the file using the command `sudo visudo`:
   * Add `jenkins ALL = NOPASSWD: ALL` _OR_ `jenkins ALL=(ALL) NOPASSWD:ALL` (See which format is used in the file)
5. `sudo passwd jenkins`, password: s0uthw4rk
6. * `sudo su - jenkins`
   * `mkdir .ssh`
   * `vim .ssh/authorized_keys` - Add in the public key from an existing server
7. `chmod 700 ~/.ssh`
8. `chmod 600 ~/.ssh/authorized_keys`
9. * Copy the deployWar2 and ssplusConfigure files from a server that already have them from `/home/jenkins/bin` to the server you're setting up, also in `/home/jenkins/bin`.
   * `sudo chmod 755 /home/jenkins/bin/`
   * `sudo chmod 754 /home/jenkins/bin/*`
   * `sudo chown -Rv jenkins /home/jenkins/bin` -- this might not be needed, but just in case
   * `sudo chgrp -Rv deploy /home/jenkins/bin`
Confirm the permissions have been correctly set:
```	
/home/jenkins:
drwxr-xr-x  2 jenkins deploy 4096 Jan 18 11:03 bin
	
/home/jenkins/bin:
-rwxr-xr--  1 jenkins deploy 581 Jan 18 10:59 deployWar2
-rwxr-xr--  1 jenkins deploy 389 Jan 18 11:02 ssplusConfigure
```
10. Check you can connect to the server from keaton as jenkins. Starting on your local machine:
* `ssh keaton`
* `sudo su - jenkins`
* `ssh [server]`
If that doesn't work, you may need to update `/etc/hosts` on keaton. If a server's ip address has changed, run `ssh-keygen -R [server]` to remove the old entry in known_hosts.