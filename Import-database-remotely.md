An example of how to copy a DB from a Production Server to QAT
============================================================

In the below example I will use PROD@AFROBANK to AFRICAN_BANK@QAT as the example

* Connect to the production server and connect to the oracle user

```bash
$ ssh nereid

$ sudo su - oracle

$ echo $ORACLE_SID

# Change the ORACLE_SID to the correct one for the db you want to export:

$ export ORACLE_SID=afrobank

# On most servers we now have the following script that will export a database for you:

$ exp-owner prod #change this to the owner of the tablespace to be exported

$ logout

# Go back to your user and move the file to your home folder:

$ sudo mv /home/oracle/prod.afrobank.2013-04-25.dmp.gz ~

$ logout
```

* Connect to europa and stop tomcat to remove its connection to the tablespace you are going to drop

```bash
$ ssh europa

$ sudo /etc/init.d/tomcat stop

$ ps -ef | grep tomcat #this is to check tomcat has actually stopped, sometimes it takes a minute
```

* Connect to the database server for QAT (in most cases this is dione):

```bash
ssh dione

$ scp nereid:~/prod.afrobank.2013-04-25.dmp.gz .

# Now move it to the oracle users home folder:

$ sudo mv prod.afrobank.2013-04-25.dmp /home/oracle/

# Log in as oracle:

$ sudo su - oracle

# If we're moving a tablespace from one name to another (as per this example prod -> african_bank) we need to 
# create a temporary tablespace (again in this example called prod. The name of the tablespace is the same as 
# the database owner that was passed as an argument to the exp-owner command on the original server):

$ sqlplus '/as sysdba'

SQL> CREATE TABLESPACE prod DATAFILE '$ORACLE_BASE/oradata/$ORACLE_SID/prod.dbf' SIZE 256m AUTOEXTEND ON EXTENT MANAGEMENT LOCAL AUTOALLOCATE SEGMENT SPACE MANAGEMENT AUTO;

SQL> CREATE USER prod IDENTIFIED BY webby12 DEFAULT TABLESPACE prod TEMPORARY TABLESPACE temp;

SQL> GRANT EXP_FULL_DATABASE TO prod;

SQL> GRANT CONNECT,RESOURCE,DBA TO prod;

SQL> EXIT;

# On some of our db servers we have a createTablespace folder containing SQL files that can be used to 
# drop/create tablespaces. If there isn't one for the tablespace you're interested in now's a good time
# to make it:

$ cd createTablespace/

$ vim createTablespace-adcorp-african_bank.sql

CREATE TABLESPACE african_bank DATAFILE '$ORACLE_BASE/oradata/$ORACLE_SID/african_bank.dbf' SIZE 256m AUTOEXTEND ON EXTENT MANAGEMENT LOCAL AUTOALLOCATE SEGMENT SPACE MANAGEMENT AUTO;

CREATE USER african_bank IDENTIFIED BY webby12 DEFAULT TABLESPACE african_bank TEMPORARY TABLESPACE temp;

GRANT EXP_FULL_DATABASE TO african_bank;

GRANT CONNECT,RESOURCE,DBA TO african_bank;

:wq

$ sqlplus '/as sysdba'

SQL> @createTablespace-adcorp-african_bank.sql

SQL> EXIT;

$ cd ~

# Now we can import the new dump file into the new tablespace. Using either 'imp-owner [tablespace]' or as below

$ gunzip prod.afrobank.2013-04-25.dmp.gz

$ imp african_bank/webby12 fromuser=prod touser=african_bank file=prod.afrobank.2013-04-25.dmp log=african_bank.2013-04-25.import.log
```

* Once the tablespace has been successfully imported, if you have moved a tablespace from one name to another you will have tablespace ownership issues. Access your newly created tablespace in SQL Developer and run the script from line 70 onwards in `sql/tablespaceOwnershipIssues.sql` then run the checks contained in that script file to ensure they have all been resolved. Once that's done you can get rid of the temporary tablespace you created. Head back to the db server and assuming you're still logged in as oracle:

```bash
$ sqlplus '/as sysdba'

SQL> DROP TABLESPACE prod INCLUDING CONTENTS AND DATAFILES;

SQL> DROP USER prod CASCADE;

SQL> EXIT;

# Back on europa (or whichever server your app is located on) you can now restart tomcat:

$ sudo /etc/init.d/tomcat start
```

* You may(/should) make sure all the db updates apply successfully as you may get errors.