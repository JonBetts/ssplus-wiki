Please note that we now have two different oracle installations.  Kari has a 12C version whereas all other servers carry a 10g database.  **You can not import a 12c db into a 10g database!!**

A few scripts to export a tablespace from a database, from a server and import it into your dev database running on a locally running VM.

The script does the following to achieve this.

1. Runs the [export-db](github.com/skillstream/ssplus/database/scripts/export-db) script on a server
2. `export-db` sets the environment up to run exp command to export a database, we 
then gzip this file.
3. We then scp the file from the server to a local directory which is shared 
with the VM with correct permissions to access this file
4. We then ssh to the VM and run a modified [imp-owner](github.com/skillstream/ssplus/database/scripts/imp-owner) script that will check;
    * if a user exits and if so enable any table locks that stop us from dropping a
      tablespace
    * drop the current tablespace and user and recreate them
    * import the tablespace
    * reset the passwords

Setup
=====

Remotely
--------
* Add the oinstall group to your user to run the exp command in export-db
script.

```bash
sudo usermod -aG oinstall $USER
```
*N.B. for some installations (e.g. on garfield) you need to add yourself to the `dba` group instead/as well*

2. Add the [export-db](https://github.com/skillstream/ssplus/blob/master/database/scripts/export-db) script to the bin folder located in your user directory.
  * modify the ORACLE_HOME to the correct oracle version for the database.

Locally
-------
* Add the following to your `~/.bash_profile`

```bash
SSPLUS_WORKSPACE="$HOME/workspace/ssplus"
export SSPLUS_WORKSPACE
```

* run below to apply the setting to your current terminal

```bash
. ~/.bash_profile
```

* Add the `imp-db`, `create-db` and `dwn-db` scripts to your path, e.g. as below or by adding the scripts folder to your `PATH`

```bash
ln -s $SSPLUS_WORKSPACE/database/scripts/create-db ~/bin/
ln -s $SSPLUS_WORKSPACE/database/scripts/imp-db ~/bin/
ln -s $SSPLUS_WORKSPACE/database/scripts/dwn-db ~/bin/
```

Execution
---------

##Method 1 - export all tables

The script is called [imp-db](https://github.com/skillstream/ssplus/blob/master/database/scripts/imp-db) and has 4 parameters

1. USERNAME
2. ORACLE_SID
3. SERVER
4. NEW USERNAME (optional)

used as follows in the workspace directory that your database resides normally, e.g. `~/workspace/ssplus/`

###[imp-db](https://github.com/skillstream/ssplus/blob/master/database/scripts/imp-db)

	imp-db northgate qat dione

or

	imp-db northgate qat dione matttest

This script actually calls two scripts which form a two stage process:

###[dwn-db](https://github.com/skillstream/ssplus/blob/master/database/scripts/dwn-db)

This is used to ssh to the server and run the script export-db which you placed 
there earlier.  Then scp the gzipped file to your machines ```/Users/Shared/tmp```

You can run this step with the following:

	dwn-db northgate qat dione

###[create-db](https://github.com/skillstream/ssplus/blob/master/database/scripts/create-db)

This is used to import the downloaded file to the VM that vagrant is running.

You can run this step with the following:

```bash
# This will run the create-db for the latest file in `/Users/Shared/tmp`
create-db

# This script will create the db as the same tablespace as the old one.
create-db northgate northgate.qat.2012-08-24.dmp.gz

create-db northgate matttest northgate.qat.2012-08-24.dmp.gz
```

##Method 2 - with control over which tables are exported
It is possible to exclude one or more database tables during the export process. 
This is particularly useful for skipping the ones which hold large volumes of data and are unlikely to be of interest on a developer's machine, thus speeding up the export, transfer and import operations. 
Some of these tables are:
 * AMENDMENTS
 * AMENDMENT_EVENTS
 * AMNDDETS
 * TIMESHEET\_GATE\_DATAS
 * TIMESHEET\_IMPORT\_EXCEPTIONS
 * TIMESHEET\_LAYERS\_IMPORTS
 * TIMESHEET\_SHIFT\_LOGS

In order to perform this selective export, you'll be running the `export-db` script directly on the server. 
To do this, launch the terminal and establish an SSH connection between your machine and the database server. For example:

	ssh dione

Execute the `export-db` script:

	./export-db qat ibm

Whenever the export process begins exporting a table you wish to skip, press _Ctrl + C_ to cancel it and move over to next table.

Once the export completes, copy it to you local machine using `scp`.
For example:

	scp dione:/home/<username>/bin/ibm.qat.2015-08-24.dmp.gz /Users/Shared/tmp

Once downloaded refer to the `create-db` above to import the file locally.

N.B. Following assumptions are made
===================================
1. If using **Method 2**, you can't use ctrl + c to cancel exporting a table e.g. amnddets or alike.
2. The user locally should also be the user remotely, although modification to the script will allow this to work.
3. This script also assumes you have the vagrant setup from current Vagrantfile on master