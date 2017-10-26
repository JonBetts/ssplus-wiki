1. Please ensure you have the following entry for _server_ in your `~/.ssh/config` file and your `/etc/hosts`
`Host server`
	`LocalForward port localhost:1521`
2. Also you will need to remove your ganymede entries from ~/.ssh/known_hosts
`ssh-keygen -R server` OR `ssh-keygen -R server.skillstream.co.uk`