Here’s how you add a new user, alex, with default home folder setup, bash shell and ssh access via RSA key:

`sudo useradd -m -s /bin/bash alex`

`sudo passwd alex # set temporary password`

`sudo su - alex`

`mkdir .ssh`

`vim .ssh/authorized_keys # paste public key`

`chmod 700 .ssh/`

`chmod 600 .ssh/authorized_keys`

`logout`

--TODO
Look at a way to do this with chef