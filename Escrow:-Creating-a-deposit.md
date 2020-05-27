# To put together an ssplus escrow deposit

Create a folder for this deposit and cd into it.
Ensure your ssplus workspace is where you want it (master is fine)

	git clone ~/workspace/ssplus
	cd ssplus
        git checkout most-recent-branch-for-client
	rm -rf .git changelog* extras/site_information

and, within the following folders, also delete

* `config/` all other clients (leave `email_config.xsl`)
* `extras/` client-specific stuff not related to this client
* `sql/` everything _except_ `clean/` and `views.sql`

Finally, use the `dwn-db` script to dump all relevant qat schemas and include those in a separate folder.

# Bundle and deposit
* Archive the deposit you've compiled, e.g.

		cd ~/Documents/escrow
		tar -czf Experis-39747-2014-06-16.tar.gz Experis-39747-2014-06-16

* Log onto [NCC Escrow Live](https://www.escrowlive.trust/)
* Navigate to 'Deposit Now'
* Select the agreement
* Fill in the form (specify `ssplus/README.markdown` as the doc location)
* Upload the tar.gz file you created