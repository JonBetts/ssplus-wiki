In case it's not clear, the app has to be decommissioned before its database, and there's no point doing one and not the other.

1. I'd start with removing the database's entry from `/etc/oratab` and `/home/bb/bbc3.10-bbpe/etc/bb-roracle.ids` - that stops it from being restarted on reboot and monitored by big brother, respectively.
1. Also take it out of the crontab for the backup user.
1. It's not a bad idea to also take a final exp dump as a snapshot of the point of decommissioning.
1. Finally, you can shutdown the database.

The above is sufficient, but to completely remove the db for good, as the oracle user,
1. Make sure the database has been shutdown.
1. Remove everything that gets created in the setup notes. Essentially that's
the database's folders under `$ORACLE_BASE/oradata` and `$ORACLE_BASE/admin`
files under `$ORACLE_HOME/dbs` which are specific to the SID of the database
there may not be any, but any respective entries in the listener.ora and tnsnames.ora files under `$ORACLE_HOME/network/admin`

I think Oracle 11 has a `DROP DATABASE` command. Would have been nice if Larry could have thought of that one in one of the previous 10 versions!
More