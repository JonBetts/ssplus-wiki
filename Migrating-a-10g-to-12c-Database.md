**Please ensure train is not symlinked to production otherwise you will need to migrate both databases over to 12c.**

### Please follow steps can all be done before the go live to below to migrate a client that has migrated over to a 12c database.

1. Establish the new database server - [Skillstream Server Migration](https://docs.google.com/spreadsheets/d/1eNnCo8JGqutPs6z7TZ4RSaMS12jwYsZL4BpmgkRfkGA/edit#gid=0)
2. [Install Oracle 12c Software on server](https://github.com/skillstream/ssplus/blob/master/database/Installing%20Oracle%2012c%20on%20CentOS.sh) or [If already installed jump to](https://github.com/skillstream/ssplus/blob/1b3bdc67af018b87abeae321ad807f83c76c7fb7/database/Installing%20Oracle%2012c%20on%20CentOS.sh#L91) 
3. ssh to the app server and try to establish a connection to the db server with 

`telnet {server} 1521`

4. Once database is imported please send details out to ssplus@skillstream.co.uk of the details for the new server and how to remove known_hosts and config settings please.  [Please see](Please see https://github.com/skillstream/ssplus/wiki/Reconnecting-to-a-newly-formatted-server)

5. Prepare [RMAN Backups](https://github.com/skillstream/ssplus/blob/master/database/Oracle-backup-config.markdown#backup) ready to use for the new database and transfer to the back up server.  **Don't enable** RMAN just yet (if the deployment is a large one with lots of db updates or even a data upload this will soon take the site down or make it slow to complete the next steps).

### Once the initial db is imported wait until the deployment to production.

1. Shutdown tomcat on production app server
2. Drop the existing tablespace and re import a fresh copy of the db
3. Start tomcat up on production app server and ensure db updates have been completed
4. Ensure [RMAN Backups](https://github.com/skillstream/ssplus/blob/master/database/Oracle-backup-config.markdown#backup) are up and running and config the back up server to pick up the new changes
5. Ensure RMAN is no longer backup up the old db on the old server and remove old backup once new backup has been made correctly.