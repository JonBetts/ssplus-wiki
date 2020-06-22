* ensure jenkins has been [setup](Setting-up-Jenkins-user.md)
* Run the [site setup script](https://github.com/skillstream/ssplus/blob/master/scripts/site-setup.sh) as per the instructions on the [markdown](https://github.com/skillstream/ssplus/blob/master/scripts/README.markdown) on the destination server to get the relevant folders ready. Make a note of the entry you need to add to `server.xml`.

* If the site uses any kind of web service copy across `/etc/.keystore.ssplus-ws`.

**Most of what follows will need to be done out of hours. (Most, not everything).**

* Shutdown tomcat on both servers.

* If the db needs migrating, create the database on the new server as per the [database setup markdown](https://github.com/skillstream/ssplus/blob/master/database/database-setup.markdown). Take an export of the tablespace on the current server, ship it across to the new one and import.    You will also need to add an entry to iptables to accept connections from the new app server on port 1521: `sudo vim /etc/iptables.rules` and then `sudo iptables-restore < /etc/iptables.rules` to ensure this change is persisted if the server ever needs a reboot.

* Migrate attachments using `rsync -avzu /sites/[site]/uploads [destination server]:/sites/[site]/uploads`. You might get a permission denied error. In that case on the destination server run `sudo chown -R [your user] /sites/[site]/uploads` then try again, if successful change the owner back to tomcat. (If this is a brand new server you may need to set up the backup process for attachments on pandora, see Ben or Matt about that).

* Copy across the relevant cron jobs and comment them out on the old server. You can see cron jobs set up under the root user using `sudo crontab -l`. You will also want to check if there is an amendment archive job, that will be under the oracle user's crontab.

* Add an entry for the site under the localhost ip address (127.0.0.1) to `/etc/hosts`.

* Add an entry into the server's nginx config: In `/etc/nginx/sites-available` you'll see the different files for each site on the server. In most cases you should just be able to copy one of the existing ones and then amend as needed for the new site. You will need to create a symlink for your new file in `/etc/nginx/sites-enabled`, then run `sudo service nginx reload`.

* Log into easyDNS and update the DNS entry.

* On the old server, comment out the site's entry in `server.xml` and then `sudo mv /sites/[site] /sites/_old`. You can now start up tomcat on the old server.

* Amend the `live_config` setting in the site's context file to `true`. (This is by default set to `false` by the site setup script to stop any emails being fired before they should).

* Update the deploy job in jenkins so going forward they get sent to the new server. Then deploy the relevant version of the app.

* Update `/etc/hosts` on phobos so that the selenium monitor now checks the site on the new server.

* Pray it all went to plan.

(You may also want to take into consideration any train sites that were on the old server).

* Update https://smon.skillstream.co.uk and google docs to reflect the changes

[Remove the certificate](https://github.com/skillstream/chef-kitchen-wiki/blob/master/certbot---remove-a-certificate.md) from auto renewal on the old server.

